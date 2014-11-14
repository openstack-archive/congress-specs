..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============================================
Get status from the cage about dataservices
=============================================

https://blueprints.launchpad.net/congress/+spec/dse-dataservices-status

Get metrics about the data services and have it available
through the api. This information will include things so that the policy engine
will know the status of the data services and when a data service is not able
to do its job correctly.


Problem description
===================

Currently each of the datasources implemented as nodes in the DSE have
2 fields for status: the time the datasource last refreshed its data and
the last error that was generated when it tried to refresh its data.
But there are additional metrics that would be
nice to have as part of the status of a datasource/DSE node, e.g. outstanding
messages, message rate, subscribers, subscriptions.



Proposed change
===============

This change will add a status() method to each DSE node that returns
a dictionary of useful information about that node.  The node will
also compute those metrics and will be configurable so that metrics
that are expensive to compute can be turned off.


Alternatives
------------

The status() information could be implemented on top of DSE instead of
directly within DSE.  But doing so would break the abstraction that
DSE provides: a message bus that takes care of message queues, message
delivery, queueing times, etc.

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

None since the API already exposes a 'status' query for each datasource.
But the implementation of that API will change to incorporate the additional
status information provided by DSE.

Security impact
---------------

None

Notifications impact
--------------------

None

Other end user impact
---------------------

None

Performance Impact
------------------

It will be possible to turn off high-cost status computation; thus, people
can decide whether each status item is worth the cost of computing it.

Other Deployer Impacts
----------------------

None

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

- Add implementations for metrics.  The decision as to whether the metric
is continually computed or computed on demand will be made on a metric-by-metric
basis.

- Add status() method to DeepSix class and return a dictionary of metrics.

Dependencies
============

None

Testing
=======

Unit tests will suffice: simulate activity in the DSE and check that the
status metrics are as expected.  Ideally would also simulate metric computation
under load.


Documentation Impact
====================

None

References
==========

None
