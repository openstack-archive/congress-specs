..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Create table publishing middleware
==========================================

https://blueprints.launchpad.net/congress/+spec/table-publishing-middleware

To lower the barrier to entry for other projects to publish data to Congress
on the (extended) DSE, create middleware that automates as much of the process
as possible.


Problem description
===================

Currently Congress periodically polls the services it is managing to get their
data.  It would be preferable if services sent data to Congress only when that
data changes.  For example, instead of Congress pulling the list of Nova's
servers every minute, Nova would send its servers to Congress whenever a
new server is created, an existing server is deleted, or an existing server
is updated.

oslo.messaging already makes it possible for projects to publish data
to other projects, but that mechanism is underutilized today.


Proposed change
===============

Solving this problem means making it even easier for other projects to send their
data to Congress than oslo.messaging does.  This change will include middleware,
perhaps made available via oslo, that publishes all changes to the underlying
database tables on the bus.  Moreover, it will
send not the entire table but rather the delta on the table that occurred.
It will be tightly integrated into existing oslo.db so that existing projects
need make no code changes; they need only include and configure the code.

There will need to be a synchronization mechanism: Congress pulls all the data
once and then needs deltas published on the bus from that point on. The deltas
sent on the bus will include timestamps that allow Congress to synchronize
the initial pull of data with the deltas.


Alternatives
------------

Another option is to have a separate effort, say oslo.publish_tables, that
is not so tightly integrated with oslo.db.  The upside to this approach is that
projects not using oslo.db (even those outside of OpenStack) could leverage
the code.  The downside to this approach is that projects already using oslo.db
would need to write and maintain a bunch of special-purpose code for publishing
their data--an approach that has seen little success til now.

Perhaps we will find that we can implement both approaches with little extra
effort, but the primary goal of this spec is the tight integration with
oslo.db.


Policy
------

None

Policy Actions
--------------

None

Data Sources
------------

None

Data model impact
-----------------

None

REST API impact
---------------

None


Security impact
---------------

None

Notifications impact
--------------------

The goal of this spec is to improve the ability of other projects to
utilize the notifications mechanism provided by oslo.messaging.


Other end user impact
---------------------

None

Performance Impact
------------------

There will be minimal performance impact on the projects that utilize this
code because only the deltas that hit the database will be published on
the bus.  Moreover, because each project can configure which tables
are published on the bus, the project owner can tune any
possible performance impact.


Other Deployer Impacts
----------------------

When configuring the middleware, we propose one key configuration option:
which tables should be published on the bus.

Developer Impact
----------------
None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <launchpad-id or None>

Other contributors:
  <launchpad-id or None>

Work Items
----------

- Write basic middleware functionality
- Integrate it into oslo.db
- Install it into an existing service, say Nova.
- Standalone documentation

Dependencies
============

enable-delta-driven-datasources: enable the datasources/DatasourceDriver
class to connect to datasources that send deltas.


Testing
=======

- Unit testing the basic functionality
- Tempest tests
  o  Verify that changes to tables configured to be published actually
  get published
  o  Verify that changes to tables configured NOT to be published do not
  get published

Documentation Impact
====================

Need documentation in oslo for the basic middleware.
Need documentation update to oslo.db


References
==========

None
