..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================================
Experimental component for a JSON data model
============================================

https://blueprints.launchpad.net/congress/+spec/json-data-model

The translation from JSON source data to tabular Congress data places the
limited resources of Congress developers in the way of the Congress users
having the flexibility to quickly try/use any data source and data field as
the need arises in their policy use cases. This spec is a proposal to
experiment with removing the translation step, thus retaining the source JSON
data model, as a promising way to overcome the limitation.

Problem description
===================

In cloud computing, most of the source data is JSON. In the Congress model,
the developers carefully understand and curate the source data to develop the
data source drivers that make the data available to end users in table form.
The translation into table model has the advantage of presenting a simpler
and more consistent data model to the user writing policy on the data.

However, with so many sources of data out there and new data being added to
existing sources through frequent microversions (e.g., Nova), it is impossible
for Congress developers to anticipate and keep up with all the data sources or
even all the data fields in a given data source the end users desire.
It often happens that as the user writes policy, the user finds out that
additional data fields or sources are needed for the policy. In that case,
the user needs to contact upstream developers, wait for them to add the data
fields or sources, then wait for the new release, then upgrade to the latest
release. All this before the end user can even test out the desired policy.

In order to better support those operators who have rapidly emerging and
evolving use cases, we need to experiment with an additional model: remove
translation and developer curation from the loop, allowing users to integrate
new data sources without code change.


Proposed change
===============

Create an experimental component of Congress (tentatively congress-json)
to allow writing policy over json data stored in PostgreSQL json data model.

Congress-CloudState would ingest the source data mostly as-is, in JSON format,
into a PostgreSQL database using the JSONB column type in PostgreSQL. The
`PostgreSQL JSON syntax
<https://hackernoon.com/how-to-query-jsonb-beginner-sheet-cheat-4da3aa5082a3>`_
(see the policy_ section for examples) would allow for rules (SQL views) to
operate directly over the JSON data.

Because no translation is needed, new data sources can be configured without
code change. For example, the following sample YAML config file configures
Congress-CloudState to poll the nova endpoint. It tells Congress-CloudState to
poll the path servers/detail, extract the results using the jsonpath
expression ``$.servers[:]``, and finally populate the nova.servers table with
the results.

::

  name: nova
  poll: 60
  authentication:
    type: keystone_password
    user: congress_access
    password: secret
  api_endpoint: https://stack.org/compute/v2.26
  tables:
    servers:
      api_path: servers/detail
      api_verb: get
      jsonpath: $.servers[:]

Architecture
------------
The architecture is fairly simple. Congress-CloudState has two major
components.

- The data ingestor polls the cloud services via API and populates the
  database with the data. The ingestor also accepts webhook notifications.
- The executor reads from the database to determine what actions to execute,
  then calls the appropriate APIs in the cloud services to execute the
  actions.

The users mostly interact directly with the database to add/remove policy
rules and query policy results.

The following diagram shows the high level components and flow of data.

::

  +----------------------------------------------------------+
  |                  Cloud Services                          |
  +--------------------------------------------------^-------+
         |                                           |
         |                                           |
  +------v--------+    +---------------+    +----------------+
  | Congress-JSON |    |               |    |  Congress-JSON |
  | Data Ingestor +---->  PostgreSQL   +---->  Executor      |
  |               |    |               |    |                |
  +---------------+    +----------^----+    +----------------+
                            |     |
                            |     |
                       +----v----------+
                       |               |
                       |    Users      |
                       |               |
                       +---------------+

Benefits
--------

- Very easy to add new data source because no translation is required. This
  `blog post
  <https://pndablog.com/2017/04/26/schema-on-write-vs-schema-on-read/>`_
  has more discussion of the pros and cons of the approach (schema-on-read).
- No code change and release needed to consume a new field in a new API
  microversion. The operator simply changes the desired microversion in
  Congress-CloudState configuration.
  This `patch <https://review.openstack.org/#/c/611516/>`_ is a recent
  example of how much time and work it takes to consume a new field previously
  not included by Congress, not to mention the delay for users waiting for a
  new release to consume a new field.
- Much is available in the way of editors, debuggers, tutorials, and online
  help for PostgreSQL language, making it easy for new users to write
  policies.
- PostgreSQL has `JSON indexing support
  <https://blog.2ndquadrant.com/nosql-postgresql-9-4-jsonb/>`_
  which allows for a wide class of performant queries.

Alternatives
------------

- Directly make changes to Congress agnostic policy engine to support JSON data

  - Here is a rough estimate of the work required to implement similar
    functionality in Congress itself.

    - Design a new version of Datalog language to support JSON. This step is
      hard. As of now, there is no concensus in the Datalog community on how
      best to do this.
    - Major changes to congress/datalog/* to process the new language.
    - Major changes to the policy engine to support the new language.
    - Changes to congress/dse2/* to accommodate JSON data.

  - Instead of attempting all the work outlined above, we propose to create
    a very simple new component which leverages PostgreSQL allow policy over
    JSON data.
    If the approach proves successful, a decision could be made in the future
    to allow congress-json to be used as a tool for extracting from JSON data
    the table data needed by other parts of Congress (e.g., agnostic, z3).

- Homegrow a policy engine supporting JSON data rather than use one
  off-the-shelf

  - It would take a tremendous amount of unnecessary development and
    maintenance work.

- Choose a datastore other than PostgreSQL

  - The following are the major requirements for the datastore

    - JSON data support
    - Widely-adopted declarative language
    - Performant (off-key) join
    - Support for rules (views)
    - Transactional write
    - Strong open source community with appropriate licensing
    - Highly-available deployment option
    - Fine-grained permissions

  - Most of the well-known NoSQL solutions do not provide performant off-key
    joins
  - MySQL/MariaDB has support for JSON data but limited indexing support
    (typically requiring the indexed value be declared as a virtual column)
  - Among all the options evaluated, PostgreSQL best satisfies the
    requirements, with the following notable features:

    - Flexible indexing for JSONB data type supporting performant off-key
      joins
    - Very strong open source community with permissive license
    - A wealth of (first party and third party) tools and online knowledge for
      learning, query writing, query debugging, and performance-tuning
    - A flexible and expressive permissions system
  - We propose to start with PostgreSQL, while leaving the option open to add
    support for other datastores down the road through a plugin architecture.


Policy
------

Here we give several examples to demonstrate how policy and rules would be
created. Each interaction is shown in both classic Congress interaction as
well as Congress-CloudState interaction for reader’s understanding.

Sample data
~~~~~~~~~~~
The nova.servers table is the collection of all the JSON documents
representing servers, each document in a row with a single column d containing
the document. Each server is represented by a JSON document as supplied by
Nova’s list servers (detailed) API. Here is a simplified version of a sample
JSON document representing a Nova server (the UUIDs have been replaced with
more readable strings for ease of illustration):

::

 {
   "id":"server-134",
   "name":"server 134",
   "status":"ACTIVE",
   "tags":[
      "production",
      "critical"
   ],
   "hostId":"host-05",
   "host_status":"ACTIVE",
   "metadata":{
      "HA_Enabled":false
   },
   "tenant_id":"tenant-52",
   "user_id":"user-830",
   "flavor": {
     "disk": 1,
     "ephemeral": 0,
     "extra_specs": {
       "hw:cpu_policy": "dedicated",
       "hw:mem_page_size": "2048"
     },
     "original_name": "m1.tiny.specs",
     "ram": 512,
     "swap": 0,
     "vcpus": 1
   }
 }


Example 1
~~~~~~~~~

- Create policy (schema).

  - Congress syntax:
    ::

     congress policy create vm_error
  - Congress-CloudState equivalent:
    ::

     CREATE SCHEMA vm_host_down; 

- Create policy rule (view) identifying those servers whose host is down.

  - Congress syntax:
    ::

     congress policy rule create vm_host_down '
       error(server_id) :-
         nova:servers(id=server_id, host_id=host_id),
         nova:hypervisors(id=host_id, state="DOWN")'

  - Congress-CloudState equivalent:
    ::

     CREATE VIEW vm_host_down.error AS
     SELECT d->>'id' AS server_id
     FROM   nova.servers
     WHERE  d->>'host_status' = 'DOWN';

    - Note on syntax: the :code:`->>` operator accesses the content of a JSON
      object field and returns the result as text.

- Query the results in the policy table.

  - Congress syntax:
    ::

     congress policy row list vm_host_down error

  - Congress-CloudState equivalent:
    ::

     SELECT * FROM vm_host_down.error;

- Create policy rule (view) identifying 'critical'-tagged servers whose host
  is down.

  - Congress syntax:
    ::

     congress policy rule create vm_host_down '
       critical(server_id) :-
         nova:servers(id=server_id, host_id=host_id),
         nova:hypervisors(id=host_id, state="DOWN"),
         nova:tags(server_id=server_id, tag="critical")'

  - Congress-CloudState equivalent:
    ::

     CREATE VIEW vm_host_down.critical AS
     SELECT d->>'id' AS server_id
     FROM   nova.servers
     WHERE  d->>'host_status' = 'DOWN'
     AND    d->'tags' ? 'critical';

    - Note on syntax:

      - The :code:`->` operator accesses the content of a JSON object field
        and returns the result as JSON. In this example, :code:`d->'tags'`
        returns the array of tags associated with each server.
      - The :code:`?` operator checks that a string is a top-level key/element
        in a JSON structure. In this example, the
        :code:`d->'tags' ? 'critical'` condition checks that the string
        'critical' is in the array of tags retrieved by :code:`d->'tags'`.

Example 2
~~~~~~~~~

- Create policy (schema).

  - Congress syntax:
    ::

     congress policy create production_stable

  - Congress-CloudState equivalent:
    ::

     CREATE SCHEMA production_stable;

- Create policy rule (view) identifying production servers using an unstable
  image.

  - Congress syntax:
    ::

     congress policy rule create production_stable '
       error(id) :-
         nova:servers(id=id, image=image_id),
         nova:tags(id=id, tag="production"),
         glance:tags(image_id=image_id, tag="unstable")'

  - Congress-CloudState equivalent:
    ::

     CREATE VIEW production_stable.error  AS
     SELECT server.d->>'id'               AS server_id,
            image.d->>'id'                AS image_id
     FROM   nova.servers server
     JOIN   glance.images image
     ON     server.d->'image'->'id' = image.d->'id'
     WHERE  (server.d->'tags' ? 'production')
     AND    (image.d->'tags' ? 'unstable');

    - Note on syntax: the join-condition
      :code:`server.d->'image'->'id' = image.d->'id'` matches the glance image
      with the server whose image ID matches the glance image ID.


Example 3
~~~~~~~~~

This example illustrates that accessing deeply nested data can be relatively
straightforward.

The following view (rule) identifies all the servers using a flavor with cpu
policy 'dedicated'. Despite the information being buried several layers deep,
it is relatively straightforward to access simply by following the structure.

::

  CREATE VIEW dedicated_servers AS
  SELECT *
  FROM   nova.servers
  WHERE  d -> 'flavor' -> 'extra_specs' ->> 'hw:cpu_policy' = 'dedicated';


Policy actions
--------------

Policy actions would work similarly to agnostic engine.
The policy writer would define a policy rule (view) specifying in each row
what action to call on which service using what parameter values.

As in agnostic engine, the new component would query the 'execute' table and
then call the appropriate client/API methods according to the rows in the
'execute' table of each policy.


Data sources
------------

A new type of data source is introduced which is configured through YAML files
and stores data in the original JSON format without translation.


Data model impact
-----------------

No impact on the data model.


REST API impact
---------------

Minimal REST API impact. The webhook model for accepting webhooks is naturally
extended to the JSON data sources when configured.


Security impact
---------------

No security impact when the experimental component is not explicitly.
When enabled, there is an additional data access which would be protected by
PostgreSQL permissions set up by Congress.


Notifications impact
--------------------

No impact.


Other end user impact
---------------------

End users will have the option of using the well-known
PostgreSQL syntax to write policy directly on the source data format,
having direct access to all the source data fields.


Performance impact
------------------

Preliminary performance testing on 100,000 records show adequate performance
on both data ingestion and query evaluation. If a query performance issues
does arise, there is a wealth of tools and knowledge for tuning query
performance in PostgreSQL.


Other deployer impact
---------------------

Additional config/commandline options to control the deployment of the new
experimental component.


Developer impact
----------------

congress-json is expected to be a very simple component. As a result,
congress-json is expected to require very minimal maintenance from
developers.


Implementation
==============

Assignee(s)
-----------

Primary assignee: ekcs

Others welcome to pick up tasks.


Work items
----------

- Database initializer
- JSON-postgres data source driver
- Integration testing


Dependencies
============

The only major new dependency is PostgreSQL, which is already used in
OpenStack deployments. Version 9.4 or higher required (Ubuntu xenial comes with
9.5). It affects only those enabling the experimental new component.


Testing
=======

Simple tempest scenarios for the new component. They may piggyback on the
existing congress-tempest-postgres jobs.


Documentation impact
====================

Standard documentation expected for the new component.


References
==========

* `Schema-on-read pros and cons
  <https://pndablog.com/2017/04/26/schema-on-write-vs-schema-on-read/>`_
* `PostgreSQL JSON operators and functions
  <https://www.postgresql.org/docs/9.4/functions-json.html>`_
* `PostgreSQL JSON syntax cheat sheet
  <https://hackernoon.com/how-to-query-jsonb-beginner-sheet-cheat-4da3aa5082a3>`_
* `JSON indexing support
  <https://blog.2ndquadrant.com/nosql-postgresql-9-4-jsonb/>`_
