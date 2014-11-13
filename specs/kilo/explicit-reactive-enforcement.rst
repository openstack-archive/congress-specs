..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Explicit reactive enforcement
==========================================

https://blueprints.launchpad.net/congress/+spec/explicit-reactive-enforcement

Enable policy writers to describe conditions under which Congress should execute
specific actions, e.g. when a network is connected to a VM that contains a
virus, disconnect it from the internet.

Problem description
===================

Currently Congress takes no direct actions to help a cloud obey
the policy that governs it.  Today Congress can identify violations of
policy but it cannot correct them.


Proposed change
===============

This change enables policy writers to include policy snippets such as
the one below to encode instructions for how Congress ought to
react to changes in the cloud.  It enables the policy writer to dictate
that Congress must execute certain actions under certain conditions.

// disconnect a VM from the network whenever it is infected by a virus
execute[disconnectNetwork(vm, network=net)] :-
  antivirus:infected(vm),
  nova:network(vm, net)



Alternatives
------------

This same kind of functionality could be written in a traditional programming
language such as Python, instead of encoded as policy.  But the benefit
of doing it as policy is that we can build analysis tools that help
policy authors understand if they have created infinite loops and other
undesirable featues.

Policy
------

See example above.

Policy Actions
--------------

The purpose of this blueprint is to implement a form of reactive enforcement.


Data Sources
------------

None


Data model impact
-----------------

None

REST API impact
---------------

None.  This functionality is leveraged by writing policy statements,
which the existing API already handles properly.


Security impact
---------------

One potential security impact is that Congress typically runs as an
administrator and thus all the actions that Congress executes will be
as an administrator as well.  Thus for the first time a security breach
into Congress means a security breach into all of the services that
Congress is permitted to execute actions on.  Though in OpenStack,
a breach of security for Congress means a breach of Keystone,
which would still give administrative rights to the other OpenStack
services anyway.


Notifications impact
--------------------

None


Other end user impact
---------------------

None

Performance Impact
------------------

None for this spec; other, dependent specs have performance impacts.


Other Deployer Impacts
----------------------

None.

Developer Impact
----------------

None.


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

- Modify trigger framework to operate on modal operators directly, e.g. so
that we can register a trigger whenever execute[x] changes.

- Set up a trigger on execute[x].  When that trigger fires, it invokes
the action on the appropriate ExecutionDriver.


Dependencies
============

* modal-operators-for-policy: enable execute[] syntax
* policy-triggers: initial version of trigger framework
* action-execution-interface: add executor driver concept to Congress


Testing
=======

This change will require unit tests for the enhanced trigger framework.

It should include tempest tests to test end-to-end functionality.


Documentation Impact
====================

This will require a new documentation section for reactive enforcement.


References
==========

None
