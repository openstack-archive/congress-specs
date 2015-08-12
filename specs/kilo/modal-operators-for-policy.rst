..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Add modal operators to policy language
==========================================

https://blueprints.launchpad.net/congress/+spec/modal-operators-for-policy

To express access control policies and reactionary enforcement policies, we
need modal operators like 'execute' and 'permit' added to the language. Modal
operators are identifiers that apply to literals or rules in the language and
hence cannot be encoded (naturally) without special syntactic support.


Problem description
===================

Example policy fragments that are enabled by this change:

1) Reactionary policies: describing what actions/API-calls Congress should
execute and when.

execute[nova:disconnectNetwork(vm, net)] :-
  nova:virtual_machine(vm),
  nova:network(vm, net),
  bad_network(vm, net)

2) Access control policies: describing when other components are permitted
to execute certain actions/API calls.

permit[nova:disconnectNetwork(vm, net)] :-
  nova:owner(vm, owner),
  role(owner, 'admin')

3) Action descriptions: describing the effects of an action

delete[nova:network(vm, net)] :-
  execute[nova:disconnectNetwork(vm, net)]


Proposed change
===============

This change requires modifying the grammar to so that the constructs above
are permitted.  To do this, we would want to make the definition for 'rule'
into something like the following.

rule ::= modal_list COLONMINUS modal_list
modal_list ::= modal (COMMA literal)*
modal ::= literal | ID LBRACKET literal RBRACKET

We also need to change compile.py:Literal class to have a field 'modal' that
is either None or is an ID (string).

We also need to change the unifier so that it checks the 'modal' field when
doing unification.

Finally we need to introduce functionality so that we can ask for all
x such that execute[x] is true in the current state.  One option is to
construct a special function in the Runtime; another option is to expand the
definition of a modal so that it can take a variable, such as shown below,
and modify the unifier and runtime.py:TopDownTheory.top_down_eval routines
appropriately.

modal ::= literal | ID LBRACKET literal RBRACKET | ID LBRACKET ID RBRACKET


Alternatives
------------

Today the way we write action descriptions is by using + and - as suffixes
on table-names to denote insert and delete respectively.  This
does not extend well to other modals like 'execute' and 'permit'.

The prototype code already in place assumes it can identify what is to be
'execute'd and what is to be 'permit'ed based on the policy the rules
reside in.  Having separate policies for implementing this functionality
is a poor solution because multiple policies ought to be able to represent
multiple policy authors' contributions to policy--a single policy ought to
be able to include reactive policy, access control policy, error policy,
and action-descriptions.

In addition, the functionality we currently use for dealing with
action-descriptions while implementing simulation uses the 'consequences'
construct to compute all of the literals inserted and deleted upon execution
of a given action.  The 'consequences' functionality is computationally
expensive in that it computes the contents of all tables in the policy.  Once
a single policy contains more than simply the action-descriptions, we cannot
afford to use the 'consequences' functionality.

Policy
------

See use cases above.

Policy Actions
--------------

N/A

Data Sources
------------

N/A

Data model impact
-----------------

None since rules are stored as strings in the DB.


REST API impact
---------------

None since policy snippets are passed as strings.


Security impact
---------------

None

Notifications impact
--------------------

None

Other end user impact
---------------------

None

Performance impact
------------------

None

Other deployer impact
---------------------

None

Developer impact
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

Work items
----------

- Modify grammar as described above
- Change parser to read in new grammar
- Change runtime to properly unify with modals
- Add modal-level queries, e.g. find all x such that execute[x]


Dependencies
============

N/A

Testing
=======

Unit tests are sufficient.  Ensure that the new syntactic constructs
can be used anywhere and that the runtime produces the proper results
when the new syntactic constructs are in place.

Documentation impact
====================

End-user documentation is not necessary for this change.  But documentation
will be necessary for the functionality that uses this change, e.g.
the simulation() docs will change to describe using insert[]/delete[] instead
of +/- suffixes.


References
==========

None
