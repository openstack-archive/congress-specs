..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Policy Library
==========================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/congress/+spec/policy-library

When Congress is first deployed, it does not perform any monitoring or
enforcement because there are no policies specified. It takes significant
investment in time and energy to learn the policy language and write the
desired policies before Congress provides any value.


Problem description
===================

To help deployers and administrators  quickly get started and see value,
we propose to include with Congress a library of useful policies that a
deployer or administrator can easily enable, disable, and customize.


Proposed change
===============


Overview
------------

The proposed changes come in two main pieces, the new policy library and the
changes to existing Congress server functionalities to support the use of
policies from the policy library.

The policy library
^^^^^^^^^^^^^^^^^^^^^

Introduce a new concept of the policy library in Congress server,
where policies are retained and manipulated but are not used by the policy
engine (for monitoring or enforcement).
Include with Congress server a collection of useful policies,
each in a YAML file. These policies automatically populate the policies
library.

To make it easy for us to populate the library and for admins to modify/create
their own library, the API/CLI/GUI should support full CRUD into the library.

* Congress server can have new config option that points to a directory of
  policies to load into the library.

  * Once the library is initialized from the files, restart of Congress server
    do not re-initialize the policy library from the files
    (empty library is assumed to be un-initialized). A new interface
    allows an administrator to trigger re-initialization from files.

    .. note:: The policy library is persisted in the database and has a
      separate existence in Congress server apart from the files in the policy
      library directory. See also discussion of the alternative model
      _`Policy library directory files as truth`.

  * Future feature: reloads changed files automatically into library

    * Will not update policies imported from library into policy engine
    * Open question:
      Should changes to the policy library through the API sync back to the
      files in the policy library directory?
    * Care must be taken to consider concurrent changes and potential conflicts
      from multiple congress server instances.

.. warning:: When Congress server policy engine service is replicated,
    different server instances must have the same
    policy library files. If not, the library will be populated based on the
    an arbitrary combination of the policy files on the different server
    instances.

* Installation

  * Standalone: installer scripts copy the default library directory from the
    Congress git repo into /etc/congress/.
  * Devstack: same as standalone


Support for using policies from the library
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We propose to add all the functionalities required to support the following
workflows for activating a policy from the policy library (i.e., creating an
(active) policy in the policy engine by importing a (inactive) policy from the
policy library).

* Activating a policy from library using CLI/API.

  #. Administrator inspects policies in library to find one they want to use.
  #. Administrator downloads the relevant policy, in YAML or JSON.
  #. Administrator modifies YAML/JSON encoding and uses it as the payload to
     create a new policy.

* Activating a policy from library using GUI.

  #. Administrator peruses policies in library
  #. Administrator clicks on an interesting policy, which brings up a textbox
     (or more sophisticated policy editor) to customize.
  #. Administrator clicks on a ‘Create’ button that creates that policy.

Below are the specific functionalities needed to support these workflows.

* A backward compatible change to the policies POST API to allow for a 'rules'
  key in the JSON/YAML payload body.

* A new optional parameter to the ``policy create`` CLI command that allows
  the administrator to import and activate a policy from a YAML file.

* A new optional parameter to the ``policy create`` CLI command that allows
  the administrator to import and activate a policy directly from the policy
  library without saving to file.

  .. note:: The policy engine rule insert code would require some careful
    reworking to ensure policy creation with rules is transactional. That is,
    if insertion fails at the third rule (say because of cross-policy
    recursion), the first and second rule insertions are undone without having
    caused any effect (triggered actions or altered query answers).

    Three main things are needed.

    * Write to database only after the insertion of all the policy rules have
      succeeded in the policy engine.

    * Trigger evaluation must be delayed until after all the rules are inserted
      successfully.

    * The policy engine must be prevented from answering queries while in the
      middle of processing a batch of rule insertions. This can be done either
      with locks or by making sure the greenthread executing the batch rule
      insertion does not yield.

* A new page in the Horizon GUI where an administrator can perform the
  following tasks.

  - browse and inspect policies in the policy library
  - active (i.e., import to policy engine) a policy from the policy library
  - customize a policy in the policy library prior to import


Alternatives
------------

This section discusses several alternative designs and implementation to
various aspects of this proposal.

Policy library strictly external to Congress
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* No concept of library in API/data model. The policy library is simply a
  directory of files the administrator can import into the policy engine.

  * Least development effort.

  * Not as usable. Administrator cannot use the existing Congress interface
    (CLI/GUI/API) to browse, inspect, customize, and activate policies in the
    policy library.

  * This version is realized by a subset of the changes proposed in the spec.
    This subset should be prioritized in the development to realize the minimal
    viable functionality described here.

Policy library directory files as truth
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Instead of database, use the policy files in the policy library directory as
  source of truth.

  * Complex in error-prone in a setup with multiple policy engines.

  * Different model from all the other persisted data in Congress server,
    adding more complexity for developers.

Library rules as disabled policy engine rules in the same database table
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Instead of storing the rules in the library policy in a separate logical
  repository as well as a separate database table from the active rules in the
  rule engine, store all the rules as rules in the rule engine, with an
  additional flag indicating which ones are active.

  * Complicates the existing data model for rules in the policy engine.

  * What is the benefit?


Policy
------

Not applicable.

Policy actions
--------------

Not applicable


Data sources
------------

Not applicable.


Data model impact
-----------------

No impact on existing tables.

One new database tables are needed:

* A ``library_policies`` table to store the policies in the policies library.
  Below are the columns.

  * ``name =
    sa.Column(sa.String(255), nullable=False, unique=True, primary_key=True)``
  * ``abbreviation = sa.Column(sa.String(5), nullable=False)``
  * ``description = sa.Column(sa.Text(), nullable=False)``
  * ``kind = sa.Column(sa.Text(), nullable=False)``
  * ``rules = sa.Column(sa.Text(), nullable=False)``

  .. note:: Because the rules in the library are accessed at the granularity of
    one entire policy, we can improve simplicity and performance by storing the
    rules as a text blob instead of as individual rows.

  .. warning:: the TEXT type is limited to 2^16 - 1 bytes in length in MySQL.
    Very large policies beyond this size are not supported. If this limit turns
    out to be too restrictive, MEDIUMTEXT or LONGTEXT type can be forced by
    specifying a longer length in the SQL Alchemy column declaration, at the
    cost of losing SQLite support (primarily used for unit testing).

Empty tables will be created using sqlalchemy migration scripts. Tables will be
populated by Congress server from policy files in the policy libraries folder.

REST API impact
---------------

Policy library methods
^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Re-initialize

  * Re-initialize policy library from the files in the policy library directory
    as set in configuration.

  .. warning:: This operation destroys the current content of the policy
    library.

  * Method type: PUT

  * Normal http response code(s): 200

  * Expected error http response code(s): 401

  * ``/v1/library``

  * Parameters which can be passed via the url: None

  * JSON schema definition for the body data if allowed: Not allowed

  * JSON schema definition for the response data if any: None

* List policies

  * List metadata for all policies in the library (id, name, desc, number of
    rules)

  * Method type: GET

  * Normal http response code(s): 200

  * Expected error http response code(s): None

  * ``/v1/library``

  * Parameters which can be passed via the url: None

  * JSON schema definition for the body data if allowed: Not allowed

  * JSON schema definition for the response data if any:

    ::

      - !Type:
        title: collection of PolicyProperties
        type: array
        items:
          type: object
          properties:
            name:
              title: Policy unique name
              type: string
              required: true
            description:
              title: Policy description
              type: string
              required: true
            kind:
              title: Policy kind
              type: string
              required: true
            abbreviation:
              title: Policy name abbreviation
              type: string
              required: false


* Show policy

  * Show specified library policy (in specified format)

  * Method type: GET

  * Normal http response code(s): 200

  * Expected error http response code(s): 401, 404

  * ``/v1/library/<policy-name>``

  * Parameters which can be passed via the url: None

  * JSON schema definition for the body data if allowed: Not allowed

  * JSON schema definition for the response data if any:
    `Policy full schema`_.

* Create policy

  * Create a new policy in library

  * Method type: POST

  * Normal http response code(s): 200

  * Expected error http response code(s): 400, 401, 409

  * ``/v1/library``

  * Parameters which can be passed via the url: None

  * JSON schema definition for the body data if allowed:
    `Policy full schema`_.

  * JSON schema definition for the response data if any:
    `Policy metadata schema without UUID`_.

* Update policy

  * Update a policy in library

  * Method type: PUT

  * Normal http response code(s): 200

  * Expected error http response code(s): 400, 401, 404, 409

  * ``/v1/library/<policy-name>``

  * Parameters which can be passed via the url:

    * format: {json, yaml} (default is json)

  * JSON schema definition for the body data if allowed:
    `Policy full schema`_.

  * JSON schema definition for the response data if any:
    `Policy metadata schema without UUID`_.

* Delete policy

  * Delete policy from library

  * Method type: DELETE

  * Normal http response code(s): 200

  * Expected error http response code(s): 401, 404

  * ``/v1/library/<policy-name>``

  * Parameters which can be passed via the url: None

  * JSON schema definition for the body data if allowed: Not allowed

  * JSON schema definition for the response data if any: None


Policy engine methods
^^^^^^^^^^^^^^^^^^^^^^^^

* Create policy

  * Create a policy in policy engine (either from input data or from library)

  * Method type: POST

  * Normal http response code(s): 200

  * Expected error http response code(s): 400, 401, 404, 409

  * ``/v1/policies``

  * Parameters which can be passed via the url:

    * library_policy=<name of policy in library>. This creates the policy
      directly from the library policy, without needing body data. A request
      where both this parameter and a body data are specified should result in
      error 400.

  * JSON schema definition for the body data if allowed: `Policy full schema`_.

  * JSON schema definition for the response data if any:
    `Policy metadata schema with UUID`_.


Common JSON Schemas
^^^^^^^^^^^^^^^^^^^^^^^^^


Policy metadata schema without UUID
++++++++++++++++++++++++++++++++++++++++++++

::

  - !Type:
    id: PolicyProperties
    title: Policy Properties
    type: object
    properties:
      name:
        title: Policy unique name
        type: string
        required: true
      description:
        title: Policy description
        type: string
        required: true
      kind:
        title: Policy kind
        type: string
        required: true
      abbreviation:
        title: Policy name abbreviation
        type: string
        required: false

.. note:: UUIDs are not used for policies in the policy library. The reason
      is gets confusing if the UUID is changed when the policy is activated,
      but also problematic if the UUID is retained in the activated policy in
      policy engine, leading to two different entities sharing the same UUID.


Policy metadata schema with UUID
++++++++++++++++++++++++++++++++++++++++++++

::

  - !Type:
    id: PolicyProperties
    title: Policy Properties
    type: object
    properties:
      id:
        title: UUID for the policy
        type: string
        required: true
      name:
        title: Policy unique name
        type: string
        required: true
      description:
        title: Policy description
        type: string
        required: false
      kind:
        title: Policy kind
        type: string
        required: false
      abbreviation:
        title: Policy name abbreviation
        type: string
        required: false


Policy full schema
++++++++++++++++++++++++++++++++++++++++++

.. note:: It is a design decision that rules in a policy library do not
      have IDs and cannot be referred to individually. Policies in the library
      are created, updated, or deleted at the level of the whole policy.

::

  - !Type:
    id: PolicyProperties
    title: Policy Properties
    type: object
    properties:
      name:
        title: Policy unique name
        type: string
        required: true
      description:
        title: Policy description
        type: string
        required: true
      kind:
        title: Policy kind
        type: string
        required: true
      abbreviation:
        title: Policy name abbreviation
        type: string
        required: false
      rules:
        title: collection of rules
        type: array
        required: false
        items:
          type: object
          properties:
            id: PolicyRule
            title: Policy rule
            type: object
            properties:
              rule:
                title: Rule definition following policy grammar
                type: string
                required: true
              name:
                title: User-friendly name
                type: string
                required: false
              comment:
                title: User-friendly comment
                type: string
                required: false


Security impact
---------------

In general, no major security impact is expected. However, below are some
security considerations.

* If policy library changes are sync'ed back to the policy library directory on
  the server, then an attacker who gains administrative privilege to Congress
  can also create and modify files in the policy library directory of the
  Congress servers.

* The ability to submit multiple rules in a policy creation API can increase
  the potential for resource exhaustion attacks.

Notifications impact
--------------------

No impact

Other end user impact
---------------------

No impact to existing work flows.

Performance impact
------------------

Minimal performance impact expected. Care must be taken to limit the
performance impact of the feature to automatically detect changes in
the policy library folder and update the policy library accordingly.

Other deployer impact
---------------------

There is minimal negative impact on deployers.

* No existing database table schemas are changed.

* New database tables are used, which would be added by alembic migration
  scripts.

Developer impact
----------------

Minimal developer impact.


Implementation
==============

Assignee(s)
-----------

Assignment to be done on launchpad.

Work items
----------

To be organized on launchpad.


Dependencies
============

No new dependencies.


Testing
=======

In addition to unit tests, straight-forward additions to the tests in
congress_tempest_tests/tests/scenarios/test_congress_basic_ops.py would
suffice.


Documentation impact
====================

Minimal impact.


References
==========

No references.
