..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Datalog aggregates
==========================================

Add aggregates sum, count, min, max, avg to the policy language.

Problem description
===================

Many policies require counting the number of VMs
with a certain property, computing the average, min, max, or sum
of certain values.  Without aggregates, these policies cannot be
expressed in Datalog.

Proposed change
===============

This change adds aggregates to the Datalog policy language.

Alternatives
------------

The alternative is to leave aggregates out of the language.  The
drawback is that some common policies would be inexpressible, but
the benefit is that the algorithms for reasoning about policy
would be simpler.  By prohibiting aggregates, we exclude the possibility
that the user wants to use aggregates at the cost of their inclusion.

Policy
------

Example: define a table that counts the number of VMs in the web tier
of each application.

app_web_size(id, count(vm)) :-
    appservice:app(id),
    appservice:webtier(id, vm)


Another option for syntax puts the aggregates into the body of the rule.
This option will likely require new syntactic restrictions on rules
to function properly.

app_web_size(id, cnt) :-
    appservice:app(id),
    appservice:webtier(id, vm),
    count(vm, cnt)

Suggestions for other syntaxes?


Policy actions
--------------
N/A

Data sources
------------
N/A

Data model impact
-----------------

The syntax for rules will be extended, but the database schema for storing
rules will not, since they are stored as strings.


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

Performance impact
------------------

There will be some performance impact in using aggregates since
they cost more to compute.  These aggregates will also make reasoning
about the policy itself (i.e. when data is unknown) more difficult.


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
  <launchpad-id or None>

Other contributors:
  <launchpad-id or None>

Work items
----------

0. Choose aggregates to implement, e.g. count, min, max, sum, avg
1. Modify congress/datalog/congress.g to accept the new statements (possibly)
2. Modify congress/datalog/compiler.g to include new datastructures for
   representing aggregates internally
3. Modify congress/datalog/topdown.py:TopDown.select() to handle aggregates.
   This will require adding in a step after computing all solutions
   that applies the aggregate function.


Dependencies
============

N/A



Testing
=======

- Unit tests will be added to congress/tests/datalog/test_nonrecur.py
- A unit test or two ensuring the API properly accepts the new syntax
  will be added to congress/tests/test_congress.py



Documentation impact
====================

Need to add explanation of new syntax constructs to docs.


References
==========

1. Overview of a Datalog language with aggregates:
http://dexter.stanford.edu/main/dexlog.html

2. Recent Stanford paper on aggregates
http://stanford.edu/~abhijeet/papers/abhijeetSARA13.pdf
