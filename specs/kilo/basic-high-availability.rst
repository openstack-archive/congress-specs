..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============================
Support basic high availability
===============================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/congress/+spec/basic-high-availability

Congress needs to support high availability (HA) for API requests so that
a client can continue to make successful API requests to Congress even if a
congress server becomes unavailable. This proposal describes a basic HA
solution that replicates the entire Congress server as-is. Each replica runs
the policy engine, contains all the table data, and runs the datasource
drivers.


Problem description
===================

Today, Congress runs as a single standalone server.  That single server is
responsible for handling all API queries and is a single point of failure.  If
the server fails, it will cause downtime for clients that integrate Congress.


Proposed change
===============

This spec proposes to
 * Replicate the entire Congress server
 * Use an off the shelf load balancer to distribute requests to the replicas
   and avoid failed replicas
 * Write API calls modify the database
 * Each replica periodically checks the database for changes to policy or
   datasources


Alternatives
------------

A more advanced design separates the policy engine from the datasources, and
replicates the policy engine N times, but uses a master-standby configuration
for the datasource driver.  This way only the master datasource driver talks
to datasources thus reducing the load on datasources.  The datasource
communicates incoming data changes to the replica over a message bus.  This
change would require more code changes to separate the engine from the
datasources and to change how the messaging bus works.

Yet another proposal is to funnel all datasource updates to a central machine
which would precompute materialized views of all tables.  This has the
advantage of giving the replicated Congress API consistency, but it relies on
a single machine to compute the materialized views, and relies on
materializing the views which stores all intermediate table content in memory,
which can consume an unmanagable amount of memory with many intermediate
tables.  This alternate proposal would also require a significant amount of
code changes.


Policy
------

None


Policy actions
--------------

None


Data sources
------------

We need to ensure that each data service (such as Nova or Neutron) can accept
and handle requests from more than one datasource driver instance at the same
time since each replica will be fetching data from each data service.  In
other words, if there are N replicas, then each data service must respond with
all the data separate N times, and the data service must be able to cope with
that higher request load.


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

None


Other end user impact
---------------------

Two API calls may return different data if a different replica serves each API
call because both the data from the datasources and the policy rules may be
out of sync between two replicas.  The rate at which each replica checks the
database for updates can limit the problem for policy rules, but data skew
will still affect the replicas.


Performance impact
------------------

This change should improve throughput for the Congress server since there can
be multiple replicas instead of a single server.  However, there may be an
impact on datasources since each replica will be requesting data from the
datasource.  The period database requests to check for updates should have a
minimal impact on performance.


Other deployer impact
---------------------

None


Developer impact
----------------

All shared state must be stored in the database, and periodically checked at
all replicas.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  ayip@vmware.com


Work items
----------

 * Add period check for database updates and add a test
 * Add a test that starts two replicas and queries both


Dependencies
============

None


Testing
=======

Start two replicas, using the same database.  Write a policy change on one
replica and check that the policy change occurs on the second replica.

Start two replicas, and kill one.  Make sure the second replica can still
serve requests.  Start first replica again and make sure it can still serve
requests.


Documentation impact
====================

We should add a description of how to configure Congress in HA mode, with a
load balancer and a shared database.


References
==========

None
