..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Publish policy results to DSE message bus
==========================================

https://blueprints.launchpad.net/congress/+spec/publish-policy-results-to-bus

Implement subscriptions for the policy engine. That is, every time an entity
on the DSE subscribes to a policy engine table, the policy engine should
publish the results of that table on the message bus. The implementation
should utilize the trigger mechanism, which is the subject of another spec.

Problem description
===================

Currently, datasource drivers obey the publish/subscribe paradigm for tables.
The policy engine subscribes to the tables it needs from the datasources,
and the datasources publish information as appropriate.

But there are several use cases when other entities on the DSE message bus
(not necessarily "datasources" per se) would like to subscribe to tables
defined within policy.

- Building proof of concepts where an external service is informed of
policy violations and reacts accordingly, implemented for the sake of
convenience as another entity on the DSE message bus.

- Interoperable policy engines that publish their monitoring results
on the bus for other policy engines to consume.


Proposed change
===============

Every time the policy engine gets a subscription request for a specific
table, e.g. 'alice_policy:error', the policy engine registers a trigger
for that trigger.  When the trigger fires, it publishes the contents of
that table on the message bus.

Every time the policy engine gets an unsubscribe request for a specific
table, the policy engine removes the trigger for that table.

The subscribe/unsubscribe functionality should be implemented within
policy/dse_policy.py:DseRuntime.

Because it is likely that the tables could be large, it makes sense to
publish deltas for those tables on the bus, just as the datasources do.
The functionality that does this for datasource drivers can be found
in datasource_driver.py:DatasourceDriver.prepush_processor.

To implement the delta publishing, we should look into creating a subclass
of dse/deepsix.py:DeepSix that includes the prepush_processor functionality and
have both DatasourceDriver and DseRuntime inherit from it instead of DeepSix.

Alternatives
------------

The policy engine could publish the entire table to the bus.  The downside
is that large tables with frequent small changes would cause a large amount
of unneeded bus traffic.  The upside would be that the subscriber might
be simpler to write if it receives the entire table.  If that turns out to
be the case, we could always build convenience functions that compute
the entire table from the deltas.


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

Performance will be impacted, but little moreso than because of the triggers.
The triggers will need to compute the delta; that delta will then simply
be published on the message bus.  Publishing is fast (especially compared
to computing the contents of tables and then their deltas).


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
  <launchpad-id or None>

Other contributors:
  <launchpad-id or None>

Work Items
----------

- Create subclass of DeepSix that includes delta publication functionality
and have DatasourceDriver and DseRuntime inherit from that subclass
instead of DeepSix

- Alter DseRuntime so that every subscribe message sets up the appropriate
trigger.

- Alter DseRuntime so that every unsubscribe message removes the appropriate
trigger.


Dependencies
============

* Requires triggers: policy-engine-triggers

Testing
=======

Non-tempest tests that subscribe to policy engine tables, cause
changes to those tables, and verify that the appropriate deltas
are sent on the bus are adequate.


Documentation Impact
====================

None required--we're just making a policy engine implement the same interface
as datasource drivers.


References
==========

N/A
