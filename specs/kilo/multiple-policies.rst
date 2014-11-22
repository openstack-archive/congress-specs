..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Example Spec - The title of your blueprint
==========================================

https://blueprints.launchpad.net/congress/+spec/multiple-policies

Currently, there are only a couple of different policies people can use.
We want to enable people to utilize multiple policies for the purpose of
modularity and encapsulation.

Problem description
===================

We want users to be able to create and delete entire policies.
We want users to be able to choose which type of policy they want
to create.
We want users to write policies that reference tables defined in
other policies as naturally as they reference tables defined within
datasources.

Proposed change
===============
Users may create/delete any number of policies and give them names.
Each of these policies look like the classification policy we have
today.

Users can insert/delete rules into each such policy
just as they do with the classification policy today.
Users may write the same kind of rules they always have, except
they can reference tables defined in any other policy in
the body of rules.

The rules are prohibited from prefixing the tablename in the head
of a rule with a policy-name.  The rules are prohibited from
recursing across theories.

1) Example
policy1:
p(x) :- policy2:q(x)

policy2:
q(x) :- nova:servers(x)

2) Non-example: do not include policy name in head of rule
policy:p(x) :- q(x)

3) Non-example: do not recurse across policies
policy1:
p(x) :- policy2:q(x)

policy2:
q(x) :- policy1:p(x)


Alternatives
------------

We're already doing this (conceptually) with the prefixes (such as
nova: and neutron:) on datasource tablenames.  This proposal simply
generalizes that idea.

Another approach is to put everything into a single theory and explicitly
prefix all tables with the module in which they are defined.
Implementationally, that is basically what we're doing, but leaving the
policies in separate datastructures so that they can leverage different theory
types.

Many other module systems are available, but this is basically Python's
module system.  Everything is visible to everything as long as you
have a reference to it.

This proposal does not include hierarchical policies (policies defined
within other policies).  While that model is sensible for a file-based
programming language, it is less clearly sensible for a restful API
programming language since there is no inherent structure.  This
proposal still allows people to use hierarchical policy *names*, e.g.
marketing:manager:alice, but there is no semantics to that hierarchy
built into the language.



Policy
------

Example with policies policy1 and policy2.  We assume we write the
following rule in policy0.

error(vm) :-
    nova:virtual_machine(vm),
    policy1:vm_property(vm),
    neutron:port(vm, src_ip),
    policy2:ip_bad(src_ip).


Policy Actions
--------------

None

Data Sources
------------

None

Data model impact
-----------------

None.  Just a change in the policy engine's implementation.

REST API impact
---------------

Each API method which is either added or changed should have the following

create_policy

* Description: create a new policy of the given name, abbreviation, and
  type
* Method: POST
* Normal http response codes: success
* HTTP errors:
    ** conflict: policy already exists
    ** bad request: type does not exist
* URL: /v1/policy/
* Parameters: name, abbreviation (for traces), type (Nonrecursive,
  Materialized)

Example:
curl -X PUT http://localhost:1789/v1/policy -d
'{"id": "test_policy",
"description": "a great policy",
"abbreviation": "test",
"type": "nonrecursive"}'

Example output:
{"id": "test_policy",
"description": "a great policy",
"abbreviation": "test",
"type": "nonrecursive",
"owner": "alice"}'


delete_policy: standard deletion operation

Example:
curl -X DELETE http://localhost:1789/v1/policy/test_policy


Security impact
---------------

None

Notifications impact
--------------------

None

Other end user impact
---------------------

Users will interact with other policies when writing rules.

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
  thinrichs@vmware.com


Work items
----------

- Add create_policy/delete_policy to congress/policy/policy.py:Runtime.
  The arguments to create_policy should include name/abbr/type, where
  type is either NonrecursiveRuleTheory or MaterializedViewTheory.

- Change compile.atom to a separate 'module' field and 'table' field to
  avoid repeatedly parsing the tablename.  The 'module' field is None if
  there is none.

- Modify top_down_eval so that at every point in the search, it
  jumps to the policy in which the table is defined (or stays in the
  current policy if 'module' is None).

- Ensure no infinite loops across theories.  We need to check
  that the graph obtained by rules that cross policy boundaries
  is non-recursive; we can ignore rules that do not cross policy
  boundaries.

- Can leave 'includes' functionality for internal implementation.
  Should not need to use it for the change above.

- Expose this functionality through the API and CLI


Dependencies
============

None

Testing
=======

Unit tests, both positive and negative.

Positive
---------
1) Test 1
policy1:
p(x) :- policy2:q(x)

policy2:
q(1)
q(2)

Query: policy1:p(x) yields {p(1), p(2)}


2) Test 2
policy1:
p(x) :- policy2:q(x)
r(1)
r(2)

policy2
q(x) :- policy1:r(x)

Query: policy1:p(x)  yields {p(1), p(2)}

3) Test 3  (namespace separation)
policy1:
p(x) :- policy2:q(x)
q(1)
q(2)

policy2
q(3)
q(4)

Query: policy1:p(x)  yields {p(3), p(4)}


Negative
----------
1) Test 1
policy1:
p(x) :- policy2:q(x)

policy2:
q(x) :- policy1:p(x)

Should throw error.


Documentation impact
====================

Need to add docs that describe new capabilities.

References
==========

None
