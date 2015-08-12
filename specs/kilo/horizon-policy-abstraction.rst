..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===========================================
Abstraction policy definitions in Horizon
===========================================

https://blueprints.launchpad.net/congress/+spec/horizon-policy-abstraction

Congress aims to provide an extensible open-source framework for governance
and regulatory compliance across any cloud services. Users can define policies
in congress by pure Datalog. But Datalog, working with data-tables, is a very
difficult and complex description for policies. Users may have some problems
to understand and deploy policies by Datalog. So this specification provides
an abstraction for policies, and shows it in an abstraction form in Horizon,
which will facilitate users to express their policies.

Problem description
====================

Datalog is not intuitive to use, even difficult for users to express their
policies who are not familiar with it. And it may cause some misinterpretation
when translating real intent to Datalog because of the complex logic.

Proposed change
===============

Congress makes the whole cloud compliant by defining violation state and action
for violation.

Policies in Congress can be expressed by BNF as below.

::

 Congress Policy ::= violation condition, “do” action for violation

So, policy abstraction is to abstract violation state and corresponding action
to make the policy more intuitive and easy to use.

By analyzing typical scenarios, violation mainly can be divided into two parts.
One is the constraint of objects’ attributes, and another is the constraint
of relationship between several objects’ attributes.
All the objects and constraints are not just a simple set of data source
tables, but they can be divided into some categories according to their
functions and relations. So users just need to choose objects they care about
without worrying about which tables they are in.

The violation condition can be expressed by BNF as below.

::

 violation condition ::=object attribute constraint (value | object attribute)
 object attribute::=object “.” attribute

For any violation state, congress will take some actions, such as monitoring,
proactive and reactive. Though congress has just realized monitor violation,
changing cloud state to make the cloud compliant is also an important function
for congress. So, policy abstraction will provide some optional reactive
actions for different objects to resolve violations.

The action for violation state can be expressed by BNF as below.

::

 action ::= (“monitor”| “proactive”| “reactive”) data

So policies in Congress can be abstracted into "name", "objects",
"violation condition", "action" and "data".

Among these, element “name” defines a marker of a policy, which is used to
be a unique identification for a policy.

Element “objects” defines all objects which are concerned by this policy.
They are not just simple display of data source tables, but a organized set
which contains the relationship between different objects, such as, "servers",
"networks", "hosts", "subnets", etc.
For example, table "statistics" will not appear as a object, but it will be
a attribute of other objects, such as, "servers", "networks".
Another example is users could choose "servers" and "networks" without caring
about what put them together ("ports", actually).

Element “violation condition” defines the state of objects' attributes
which can produce violation, and the constraint will include comparison,
arithmetic and some predefined relationship/functions, such as, "same_group".

Element “action” defines the action needs to take for this policy,
and actions will include "proactive", "monitoring" and some specific actions,
such as, "remove", "delete". All these actions depend on the ability of
underlying components.

Element “data” defines the information gotten or needed when executing the
action, for example, when monitoring a servers violation, users can define
"data" as servers' name to be a return parameters.

That is, Congress UI will provide many elements of policies as drop-down lists,
and the combination of these elements will form various policies. All these
elements are from the summary of typical scenarios and the ability of
underlying components.
Users need to choose which one can match their needs.
Then UI will translate the information in UI into Datalog.

Alternatives
------------

N/A

Policy
-------

There is one example to express typical policy by abstraction form in Horizon.

Example: every network connected to a VM must either be public or
owned by someone in the same group as the VM.

For this example, users care about "servers" and "networks", so users will
choose these two objects from a drop-down list.
After users decide the objects, corresponding optional violation state will
be decided, which will include these two objects’ attributes and some
predefined relationship, so users can choose "not same_group" and choose who
are not in the same group. All the choices will be show as drop-down lists,
too. And users need to choose the action and data for this violation.
For example, users choose "monitoring", attributes of servers and networks
will appear in "data".

In this policy, users can create a policy as below.

+----------+----------+-----------------------------------------------+------------+--------------+
|   name   |  objects |             violation state                   |  action    |     data     |
+----------+----------+-----------------------------------------------+------------+--------------+
| policy_1 | servers  |not equal(networks.share, public)              | monitoring | servers.name |
|          | networks |not same_group(servers.tenant, networks.tenant)|            |              |
+----------+----------+-----------------------------------------------+------------+--------------+

Policy Actions
--------------
The action can be monitoring, proactive or some execute actions
which can make the cloud compliant.

Data Source
-----------

N/A

Data model impact
-----------------

N/A

REST API impact
---------------

N/A

Security impact
---------------

All parameters inputted by users need satisfy predefined standard, for example,
if values inputted in "violation condition" in reasonable range
(e.g. 0-100% for CPU utilization).

Notifications impact
--------------------

N/A

Other end user impact
---------------------

End users can be able to write policies in Horizon and use some drop-down lists
and some simple inputs to create a policy. Then Horizon will translate the
information in UI into Datalog, which will be processed in Congress.

Performance impact
------------------

N/A

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
 Yali Zhang

Other contributors:
 Jim Xu; Yinben Xia

Work items
-----------


- Abstraction form to write policies rules and actions for policies.
- Build mapping relationship between abstraction form and Datalog,
  so users can write a policy in UI other than Datalog.
- Pass information from Horizon to Congress to finish the policy creation.

Dependencies
============

N/A

Testing
=======

Need to be tested with a variety of scenarios.

Documentation impact
====================

Add instructions for policy abstraction in UI.

References
==========

N/A
