..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================================================================
Translation of Congress Table to Neutron Group-based Policy Tables
==================================================================


https://blueprints.launchpad.net/congress/+spec/congress-gbp-translation

This specification describes how to integrate Congress Policy with Neutron
Group-based Policy (GBP).


Problem description
===================
Congress provides a mechanism to allow OpenStack clients to define policy
to be applied across all OpenStack components including networking.
Neutron Group-based policy provides a high level abstraction for defining
network connectivity between groups of endpoints.

It is desirable to integrate Congress and GBP so that Congress can
monitor and enforce GBP policies. A Congress Reachability table
can be used to specify connectivity policy between endpoints. This table
can be translated into a set of tables that represent GBP entities, such as
Policy Target Groups, Policy Rules, Policy Classifier, etc.



Proposed change
===============

GBP can be represented by these tables:

* Endpoints (endpoint_id)
* Endpoint_group (endpoint_id, endpoint_group_id)
* Classifiers (classifier_id, port, protocol, direction)
* Classifier_group (classifier_id, classifer_group_id)
* Actions (action_id, action_type, action_value)
* Action_group (action_id, action_group_id)
* Policy_rule (policy_rule_id, classifier_group_id, action_group_id)
* Contracts (contract_id, policy_rule_id)
* PolicyInstance (endpoint_group_id, relation, contract_id)


A Congress Reachability policy table may be defined to form
a policy between two groups of endpoints:

Reachable (id, group1, group2, src_port, dst_port)

The goal is to translate from the input Reachable policy table to
the output GBP tables using the Congress policy language using
functions such as:

PolicyInstance(group_id1, relation, contract_id) :-
  reachable(contract_id, group_id1, group_id2, x, y),
  producer_relation(relation)




Alternatives
------------

None.

Policy
------

An example of such a policy written using Congress datalog syntax
is shown below for two groups, tier 1 and tier 2, to communicate
bidirectionally on port 80.

Operator Input Data (from operator or cloud management system)

Tier Membership (tier_id, vm_id)

     (1, 100)
     (1, 101)
     (2, 102)

Policy Input Data

Tiers(tier_id)
     (1)
     (2)

Reachability policy table

Reachable (id, src, dst, src_port, dst_port)
    (10, tier1, tier2, \*, 80)

    (11, tier2, tier1, \*, 80)


Policy Actions
--------------

These tables allow Congress to monitor and enforce GBP policies.


Data Sources
------------

Neutron Group-based Policy. Details of GBP can be found here:

https://wiki.openstack.org/wiki/GroupBasedPolicy/StackForge/repos



Data model impact
-----------------

None



REST API impact
---------------

None.


Security impact
---------------

None.


Notifications impact
--------------------

None.

Other end user impact
---------------------

None.


Performance impact
------------------

None.

Other deployer impact
---------------------

None.


Developer impact
----------------

None.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  louis.fourie, alex.yip, cathy.zhang

Other contributors:


Work items
----------

* Define translation functions.

* Implement the new constant tables and function tables to perform the
  translation.


Dependencies
============

* This is dependent on the implementation of a GBP data-source driver for
  Congress.



Testing
=======

Some sample input tables will be created and the translation verified by
checking the contents of the output trigger tables.

Documentation impact
====================

All translation details will be documented.


References
==========


* Juno Mid-cycle Policy Summit
  https://etherpad.openstack.org/p/juno-midcycle-policy-summit

