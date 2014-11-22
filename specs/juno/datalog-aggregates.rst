..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Support for using Aggregates in Datalog
==========================================

https://blueprints.launchpad.net/congress/+spec/datalog-aggregates


Aggregates in datalog are required for writing policies involving multiple
records in a single ATOM.

Problem description
===================

* Consider a policy statement "The maximum number of ports on the Subnet X
  belonging to tenant Y should not exceed N"
  To realize this policy, we need to use aggregations like COUNT, MAX, SUM etc.
  which span across multiple records of the table.
* The current Datalog support in Congress does not allow writing policies with
  such aggregates.
* Also there is no way to simulate aggregations using existing Datalog support.
* When a policy engine subscribes to an aggregate on a data source table,
  the aggregates must be updated every time there is a change in the
  corresponding table.


Proposed change
===============

The aggregate support for Datalog allows writing policy atoms of the following
form:

Example policies involving aggregates:

1. Restriction on number of ports on a subnet
error(subnet) :- neutron:networks.subnets(subnet, port),
neutron:ports(port, ip), count(num_port, port), not gt(num_port, 10)

2. Restriction on average CPU Load on a host
error(host) :- ceilometer:meters(host, cpu_util), nova:servers(host, vm_host),
average(avg_cpu, vm_host, cpu_util), not gt(avg_cpu, 80)
average(x, y, z) :- sum(z, cpu_util), count(y, vm_host), div(x, z, y)

(Note: These are tentative policy statements and may be subject to change as
the design/implementation progresses)

* The aggregates like "SUM", "COUNT" must be defined as ANTLR Terms and
  Congress must be able to parse and compile these terms.

The initial implementation will use SUM and COUNT as prototype aggregations.
Other similar aggregates can be considered as enhancements to be taken up
at a later date.

* The runtime execution should be able to update tables corresponding to the
  aggregate terms with computed values


Alternatives
------------

N/A


Data model impact
-----------------

No data model changes are expected.


REST API impact
---------------

This change does not impact REST APIs as the changes involved will only extend
the policy syntax which is fed as input to the policy APIs

Security impact
---------------

None.


Notifications impact
--------------------

None

Other end user impact
---------------------

End user will be able to write policies involving aggregates.

Performance impact
------------------

Evaluation of policies involving aggregates may involve additional processing
and thereby impacting performance.

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
  Madhu Mohan (mmohan@mvista.com)

Work items
----------

* Add new terms in Congress.g to allow ANTLR to recognize aggregate keywords
* Extend parser/compiler to accept aggregate keywords as part of the rule
  syntax and validate
* Extend runtime to compute aggregate values and update the corresponding
  tables
* Add sufficient tests to validate support for aggregates

Dependencies
============

None.

Testing
=======

* Add test cases involving aggregates
* Add Tempest tests
* Create sample policies to test some real scenarios

Documentation impact
====================

* Docs for external consumption would need to be updated with explanation on
  writing aggregate-based policies

References
==========

1. http://logic.stanford.edu/reports/LG-2012-01.pdf
