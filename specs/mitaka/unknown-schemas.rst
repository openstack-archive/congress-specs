..

==========================================
Policy Support for Unknown Table Schemas
==========================================

The new distributed architecture requires the policy engine to
handle the case when the schema for some datasource drivers are
unknown.  Today the policy engine assumes the schemas for all
datasource drivers are known at load-time.  This spec outlines
a mechanism for supporting unknown schemas.


Problem description
===================

For the new distributed architecture, the policy engine will not know
the schema for all the datasources at the time rules are loaded from the
database. This is currently problematic because column-references are
compiled away at the time rules are loaded from the database, and that
compilation procedure requires the schema.  Thus in the new architecture,
the policy engine will crash when it tries to load policy rules that
contain column references.


Proposed change
===============

The fix to this problem is to enable the policy engine to load rules
that include column references without compiling away those column references.
That is, the main reason to compile away column references is that the
internal datastructures for representing rules cannot represent those
column references natively, and hence, the column references must be
removed at load-time.  The first task is to extend the internal
datastructures in compile.py so they can represent named-columns.

The second reason column references get compiled away is that they cannot
be evaluated (even semantically) without the schema.  The second task
is to extend the run-time capabilities of the policy engine so that
rules can be disabled without being deleted.  A disabled rule will
be completely hidden from the evaluation engine when answering queries
yet will still be visible but marked as "disabled" when users view
the rules.

Every time a new rule is inserted, if its schema is unknown, that
rule must be disabled.  Moreover, every rule using a table dependent
on the table in the head of that rule must be disabled.  Similarly
for deletion except that deletion can cause other rules to be enabled.

Every time the schema changes, all rules impacted by that schema
change should be checked for consistency, and disabled rules
must be enabled once all schemas are known.  Once a rule
is enabled, the column references are compiled away.

If a 2nd schema arrives (unequal to the first), the policy engine
must check for consistency and recompile any rules whose schema
may have changed.

For example, if the following rule is inserted before the schemas
for nova-servers and neutron-networks is known, the rule will
be disabled since it has column references.

p(x, z) :- nova:servers(id=x, network=y), neutron:networks(id=y, status=z)

Then when the nova schema becomes known this rule is validated
against that schema but is not enabled because the neutron schema
is unknown.

Finally when the neutron schema becomes known, the column references
are compiled away and the rule is officially enabled.


Alternatives
------------

Instead of disabling rules, another option is to modify the
evaluation engine to do a best-effort query evaluation.  The evaluation
algorithms themselves would know about column-references, and would
attempt to operate even if the schema was unknown.

The downside to this alternative is that the rules are actually
semantically ambiguous, and hence the result of evaluation has
unknown semantic value.


Policy
------

N/A

Policy actions
--------------

N/A

Data sources
------------

N/A

Data model impact
-----------------

No database modifications are required.


REST API impact
---------------

The Rule object will have an additional boolean field representing whether
or not the rule is disabled.


Security impact
---------------

N/A


Notifications impact
--------------------

N/A

Other end user impact
---------------------

N/A

Performance impact
------------------

Rule inserts could now be slower since if the rule inserted gets disabled,
that could cause many other rules to be disabled.

Rule deletions likewise could cause many policy rules to be enabled.

Schema updates are expensive because the policy engine must do consistency
checks on all rules that are relevant, and potentially re-compile rules.


Other deployer impact
---------------------

N/A

Developer impact
----------------

N/A

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  thinrichs

Other contributors:
  <launchpad-id or None>

Work items
----------


1. Modify compile.py datastructures to natively represent column
references.  Include a 'disabled' flag.
2. Modify query evaluation engine to ignore disabled rules
3. Modify triggers to ignore disabled tables
4. Enable/disable rules on insert/delete/set-schema
- Write dependency analysis routine to compute the rules/tables
that are disabled once a given table is disabled.
Likely to need a datastructure that tracks disabled tables.
- Modify update routine to do schema check and enable/disable rules
as appropriate using the dependency analysis.
- Modify set-schema to appropriately enable/disable rules
May want to add field to rules that say which tables the compilation
was dependent on.


Dependencies
============

N/A

Testing
=======

Unit test coverage should be mostly adequate.

Only real need for tempest tests would be testing the startup of Congress,
but that is not supported with tempest.


Documentation impact
====================

N/A

References
==========

The need for this spec was discussed at the Liberty Midcycle Sprint.
https://etherpad.openstack.org/p/congress-liberty-sprint


