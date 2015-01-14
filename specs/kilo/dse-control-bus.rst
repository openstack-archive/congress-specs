..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

======================
Add Control Bus to DSE
======================

https://blueprints.launchpad.net/congress/+spec/dse-control-bus

Nodes participating on the data services engine (DSE) bus need a means of
discovering each other and their exposed data types.  Currently a periodic
broadcast mechanism is used for this purpose.  While effective, these
broadcasts are inefficient, and complicate debugging.

Instead of using the exiting broadcast-based discovery mechanism, nodes on
the DSE should publish events on a well-known "collections registry,” that every
node subscribes to.


Problem description
===================

Distribution of DSE endpoint information is currently performed using an
inefficient broadcast mechanism.  Instead, this information should be
delivered only when changes occur, or when specifically requested.


Proposed change
===============

* All instances of deepSix will subscribe to a well-known table,
  which we will refer to as the ‘collection registry’.
* When the data indexes exposed by a deepSix instance changes (including
  instance startup), the instance will publish an update to the collection
  registry.
* Each deepsix instance will maintain a list of it’s data_indexes that it has
  a desired subscription for. The instance will do a lookup in it’s local
  copy of the collection registry to find associated endpoints to send 
  subscription requests to.  As changes occur to the collection registry,
  each instance will re-evaluate it’s own subscriptions against these
  updates.
* There is a collection registry per instance of the DSE. In a hierarchical
  DSE, there could be multiple registries.  
* The periodic broadcast of data_index information can be removed after
  these changes.


Alternatives
------------

We can continue to use the existing mechanism as an alternative, but
difficulties with inefficiencies and complex debugging will grow substantially
as the number of nodes on the bus increases.


Policy
------

N/A


Policy Actions
--------------

N/A


Data Sources
------------

N/A


Data model impact
-----------------

N/A


REST API impact
---------------

N/A


Security impact
---------------

N/A


Notifications impact
--------------------

N/A


Other end user impact
---------------------

N/A


Performance Impact
------------------

At runtime, this change will reduce the time it takes to distribute
information about new nodes in the system.  It will also reduce the overall
"chatter" on the bus by eliminating unnecessary broadcasts.  We anticipate
the reduction in chatter will greatly simplify debugging the message bus.


Other Deployer Impacts
----------------------

N/A


Developer Impact
----------------

N/A


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  None

Other contributors:
  None


Work Items
----------

TBD


Dependencies
============

N/A


Testing
=======

The existing testing framework is sufficient to test to change.


Documentation Impact
====================

N/A


References
==========

N/A
