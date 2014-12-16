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

We will create a trigger-registry interface that enables a programmer
to dictate a function to run each time the contents of a given
table changes.


Programmer's perspective
------------------------
The functions that a programmer will register have a signature like
the one that follows.

def respond_to_trigger(oldtable, newtable, delta)

The programmer will register this handler on a particular theory
and a particular table.  In this example, we register the
function above so it runs each time the table "p" changes in
the policy/theory "th".

# instance of congress/policy/runtime.py:Theory
th = engine.policy("alice_policy")
# register 'respond_to_trigger' for table 'p' on that policy
th.register_trigger("p", respond_to_trigger)

From the programmer's point of view, each time the contents of
table "p" changes in the theory named "alice_policy", the function
respond_to_trigger is run and given as arguments the original
table, the new table, and the difference in the two.


Implementation
---------------
The function 'register_trigger' will be implemented in the
congress/policy/runtime.py:Theory class.  We will also have
an 'unregister_trigger' function call.  The Theory class will
be augmented to contain a hash table 'self.triggers' that
maps each table name to the set of functions that have
been registered for that table.  'register_trigger' and
'unregister_trigger' change the contents of that hash table.

Then changes will be made to NonrecursiveRuleTheory,
MaterializedViewTheory, and Database to implement the trigger:
each time update() is called, the theory will
(i) compute the contents of each table that has a trigger registered,
(ii) apply the usual update() logic,
(iii) compute the contents of each table with a registered trigger (again),
(iv) compute the deltas on each of the tables with triggers
(v) for each table t with non-empty delta, invoke all of the triggers
registered for t giving the arguments (i), (iii), (iv)

Obviously triggers can be expensive because they require 2 queries
each time data is changed.  Ideally, we would utilize policy analysis
to only query those tables when the data being update might possibly
change the contents of those tables.

For example,  suppose we registered a trigger for table p, and we
had 2 rules:

p(x) :- q(x)
r(x) :- t(x)

Updates to table t could never change the contents of p, but updates
to table q COULD change the contents of p.  Hence, the trigger
implementation would run queries on p only when the incoming update
changes q.


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
