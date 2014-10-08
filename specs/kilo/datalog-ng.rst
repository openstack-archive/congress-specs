..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========
Datalog-ng
==========

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/congress/+spec/datalog-ng

Congress needs an easy-to-use declarative language for expressing policy. This
design specification describes Datalog-ng, a version of standard Datalog that
has some extra features that are required for data querying and manipulation in
general and declarative networking in particular.


Problem description
===================

Congress needs an expressive policy language that is both easy-to-use as well as
efficient for information processing and declarative networking. Datalog is a
declarative language that provides a higher-level abstraction for querying
graphs and relational structures. It defines an efficient, recursive, query
execution and incremental data update mechanisms based on the relational model.
This design specification specifies a formal EBNF grammar for Datalog-ng, an
extension of Datalog that 1) enables tables to be referenced more easily in
queries, 2) defines syntax for adding and removing facts from the policy
database, 3) defines syntax for rules, queries, and constraints.


Proposed change
===============

This specification proposes a generalization of Datalog, called Datalog-ng,
which is suitable for declarative networking. The grammar will be defined in
Extended Backus-Naur Form (EBNF) [2]. An EBNF specification is platform-neutral,
and formally defines the syntax of a language. Note that the full expressive
power of EBNF will NOT be used; this enables access to a wider variety of tools
that may not be able to understand all of the EBNF standard. Hence, a wider
variety of tools can be used to implement a parser, an interpreter, and/or a
compiler. See [3] for more information on the benefits of formally defining a
grammar.

Change #1:  Table Referencing
[1] says: “Conceptually, Datalog describes policy in terms of a collection of
tables.” Tables are a simple way of conveying information, and lend themselves
to querying, editing, and reporting. Policy rules can be thought of as how input
from one or more tables can be transformed into output in one or more tables.
Tables are full-fledged objects, so this enables us to not only reuse tables
(i.e., the actual data), but also reuse the policies that created the data.
However, tables that have a large number of columns are hard to use, since there
is no simple way to reference their name, or even know the number of columns
they have. In addition, there is no way for a policy to know about changes in
the table, so it would likely break as well.

Change #2:  Fact Semantics
Datalog (and Datalog-ng) work by adding facts into, and removing facts from, the
database. However, there is no standard way for a policy author to introduce
semantics associated with these operations. For example, if a fact is removed by
one policy rule, it could adversely affect other policy rules that were going to
use that rule.

Change #3:  Query and Rule Semantics
Formal semantics need to be added to standardize how queries and rules are
expressed and evaluated. Currently, there is no query semantics, but this is
very easy to add.

Change #4:  Constraints
Currently, there is no way to express constraints in Datalog. For example, if
table A has four columns, and if one of the columns must not be null, then there
is no way to check that an insert of three values into a row will fail or not.
Similarly, if there are semantic constraints (e.g., row #3 must use one of a set
of enumerated values), or data type constraints, incorrect entries will either
be very hard or impossible to check, and will fail when applied. This may be
delayed, due to its difficulty and potential to introduce problems.

Change #5:  Safety
Currently, there is no way to guarantee rule safety. For example, the following
queries are NOT safe:
   foo(X, Y, Z) :- rel1(X, y) & X < Z
   foo(X, Y, Z) :- rel1(X, Y) & NOT rel2(X, Y, Z)
   foo(X, Y) :- rel1(X)

Change #6:  Grammatical Improvements
Datalog is powerful, but somewhat hard to use. A set of “syntactical sugar” will
be defined to make Datalog easier to use, especially for the native Python
developer.

Alternatives
------------

N/


Policy
------

Traditional policy rules take the form of statements that have either two or
three clauses [5]; they are called ECA (event-condition-action) and CA
(condition-action):
  ON <event-clause> IF <condition-clause> THEN <action-clause>
or
  IF <condition-clause> THEN <action-clause>
In both ECA and CA, each clause can be a Boolean combination of atoms. However,
there are other types of policy rules:
•	Goal policies
•	Utility functions
•	Promises
[6] covers the first two, and [7] is the latest of Mark’s publications on
promise theory. All three of the above are different in form and function than
ECA and CA policy rules. Datalog-ng can model the intent of most of these forms
of policy rules, which is what is needed in Congress – the ability to
declaratively specify intent.


Policy Actions
--------------

Uses of Congress Policy Rules
Possible candidates include:
* Monitoring
* Reporting (including filtering for selected values, so the user is not
    inundated with data)
* Configuring a device reactively (e.g., a threshold was violated)
* Configuring a device proactively (e.g., trending analysis predicts that a
    threshold will be violated in the future)
The first three are straightforward; the latter may be pushed beyond the Kilo
release.

Policy Rule Implementation Alternatives
The advantages of Datalog are that it is a declarative subset of first-order
logic. Declarative languages express the logic in a task without specifying the
flow of control to perform the task. First-order logic is a formal system of
logic in which each statement consists of a subject and a predicate. A predicate
can only refer to a single subject. Sentences are combined and manipulated using
the same rules as those used in Boolean algebra. Two quantifiers exist: “for
all” and “for some” (higher-order logics have additional quantifiers, such as
“for every property of an object”).

Datalog is thus more powerful than simple propositional logic, but not as
powerful as first-order logic. However, it provides a combination of power and
simplicity that is hard to match.



Data Sources
------------

This section will describe where the data is coming from. It will define
exemplar projects that produce these data.

Data Sinks
This section will describe who is consuming the data. It will define
exemplar
projects that consume these data.

Future Extensions
Since Datalog-ng is based on a subset of first-order logic, it also provides
formal reasoning and analysis. This is most likely the subject for the next
version of this specification, but it should be kept in mind so that we do not
limit this feature in any way.


Data model impact
-----------------

N/A


REST API impact
---------------

N/A


Security impact
---------------

Policy can contain the proverbial “keys to the kingdom”. So, if someone hacks
their way into the system and can start issuing policies, game over. Therefore,
some type of access control should be used with policy-based systems.


Notifications impact
--------------------

N/A


Other end user impact
---------------------

Datalog-ng is intended for Developers and Administrators, not End Users.


Performance Impact
------------------

N/A


Other Deployer Impacts
----------------------

For this to work in a secure manner, I really think we need to operate in a
secure environment, such as role-based access control (RBAC).


Developer Impact
----------------

Implementation will provide policy writers with a richer set of options


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  straz

Other contributors:
  thinrichs, sarob

Work Items
----------

The following is a short description of how the above changes will be addressed
in this specification.

Change #1:  Table Referencing
Table referencing will be supported by introducing namespaces into the policy
grammar, as well as supporting syntax, to allow tables and table elements to be
referred to by name and/or relative position.

Change #2:  Fact Semantics
New syntax will be added to differentiate between adding and removing a fact.
This will enable optional rules to identify these operations and perform
additional tasks, if required.

Change #3:  Query and Rule Semantics
This change involves adding dedicated syntax for differentiating between rules
and queries.

Change #4:  Constraints
There are many types of constraints that can be enforced. The first set of
examples comes from relational theory. For the following definitions, assume
that a table represents an entity, such as a router or a network. Note that
these types of constraints are critical for being able to safely reference
different tables from different namespaces.
* Entity integrity: a row has a unique identifier, called a primary key. The
    primary key is unique and not null. This enables each row in the entity to
    be identified.
* Referential integrity: sometimes, tables reference other tables. A foreign
    key is a set of attributes in one table that uniquely identifies a row of
    another table a primary key from one table that appears in another table.
    Referential integrity defines the dependency of one table on another table.
* Value constraints: data has constraint(s) on the value(s) that it can take
    on. For example, a physical chassis can only be mounted in a single rack.
* Domain constraints: entity attributes in a given domain are restricted in
    one or more ways. For example, the sum of two values from two columns in the
    same row is less than or equal to another value. This generally include data
     type, data value, and the defaulting of values.

The second set of examples comes from applications of security. In this view, a
constraint is an assertion that needs to be satisfied. The typical example is
that a policy should be able to specify the resources that a user (or
application) can access, as well as the set of operations that this user can
perform on that set of resources. For example, specifying permissions for all
sub-directories and files under a given directory in intractable in Datalog,
because the set of resources could be infinite, and Datalog does not have
function symbols.

This will be defined using constraint domains, and is an optional part of the
Datalog-ng language. It may be pushed out beyond Kilo if it becomes too
difficult.

Change #5:  Safety
A rule is safe if all of the variables in the head of the rule also occur in a
positive, non-arithmetic literal in the body of the rule. This guarantees
termination of the rule. The grammar should include safety checks to protect
developers from themselves. :-)

Change #6:  Grammatical Improvements
A set of grammatical improvements will be defined to simplify the use of
Datalog-ng, and especially to make its syntax friendlier to Python developers.
Examples include more recognizable comments (e.g., familiar “//” or “/*..*/”
instead of the native Datalog ‘%’), the ability to use single and/or double
quotes, and English equivalents to some commands (e.g., ‘!’ or ‘NOT’ or ‘not’).


Dependencies
============

This spec may be broken up into multiple blueprints for implementation. The list
of spec and/or blueprints will be listed at the top of this spec.


Testing
=======

N/A


Documentation Impact
====================

N/A


References
==========

The following are references for this specification.
[1]	Congress Design, http://goo.gl/YFd2Fr
[2]	ISO/IEC, “Information technology – Syntactic metalanguage – Extended BNF”,
    14977, 12/15/1996
[3]	J. Strassner, “A Gentle Introduction to EBNF”, TBD
[4]	Congress Policy Workshop, TBD
[5]	J. Strassner, “Policy Based Network Management”, Morgan Kaufman Publishing,
    978-1558608597, 9/2003
[6]	J. Strassner, J. Kephart, “Autonomic Systems and Networks – Theory and
    Practice”, NOMS 2006 Tutorial
[7]	M. Burgess, J. Bergstra, “Promise Theory – An Introduction”, xtAxis Press,
    2014

