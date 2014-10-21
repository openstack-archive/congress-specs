..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Policy Engine Triggers
==========================================

Blueprint: https://blueprints.launchpad.net/congress/+spec/policy-engine-trigger

Currently there is no mechanism for treating changes to
policy-defined tables as events (aka triggers).  Treating changes to
policy-defined tables as events is useful because
it enables us to, for example,
(i) publish those changes on the message bus
(ii) set up reactive enforcement, either written in code or
in policy,
(iii) kicking off translations to other policy
engines.

Problem description
===================

This spec aims to provide a programmatic interface for triggers.
A programmer will register an event handler to run whenever
a change to a table occurs.  The framework will then
periodically (e.g. whenever inserts/deletes occur)
compute updates to tables and run the registered event handlers.

Proposed change
===============

To each Theory class or subclass in runtime.py we will add an
interface for event-handler registration, e.g.

Theory.register_handler(tablename, function)

To each Theory we will add code that implements all the triggers
that have been registered.


Alternatives
------------

Programmers could write their own trigger-logic for each feature
that needs this functionality.  The benefit would be that the
programmer can customize the functionality to fit exactly her
purpose.  The drawback would be that the user would need to
understand the internals of all the Theory subclasses.

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

None

Other end user impact
---------------------

None


Performance Impact
------------------

This added functionality may or may not come with a performance cost.

NonrecursiveRuleTheory will have a performance cost because currently
we do not evaluate a table on insert/delete, but after the change
we will need to do more query evaluation.

MaterializedViewTheory will incur no performance cost because it already
evaluates all tables at every insert/delete.


Other Deployer Impacts
----------------------
None

Developer Impact
----------------

None outside of Congress.  Developers can ignore the new interface
if they want.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Tim Hinrichs

Work Items
----------

- Add event-handler registry to theory base class
- Add implementation of event-handler to all theory subclasses


Dependencies
============

None


Testing
=======

Unit tests: register event handler, insert/delete policy data, check if
event handler actually executed


Documentation Impact
====================

None

References
==========

None
