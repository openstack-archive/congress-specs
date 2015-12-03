..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============================================================
Add availability zones to nova datasource driver
===============================================================

https://blueprints.launchpad.net/congress/+spec/add-az-to-nova-driver

This specification presents to extend the current nova datasource driver for
supporting availability zone and its fields available from nova.

Problem description
===================

The current nova provides avaiability zones for server and host. A host is a
part of availability zone and server can be booted on the availability zone.
However, the current nova datasource driver does not support it.
This availability zone are useful for creating any policy related to server
location.
For example, we can specify a policy for checking status of any VMs located in
the specific availability zone. In addition, we can specify an action policy
for migrating some VMs from availability zone A to availability zone B.

Recently, availability zone was added to servers using
OS-EXT-AZ:availability_zone [1].
However, it is hard to make full hierarchy among availability zone, hosts, and
servers and lack of flexiblity and extensibility. If we add the availability
zone to nova data source driver, it will be more flexible and extensible to use
more fields available from the availability zone such as zone state.

Proposed change
===============

Basically, we can extend the curret Nova datasource driver
(datasources/nova_driver.py) by adding the following a translator and
two fields.

* availabilty_zones_translator

  * zone_name
  * zone_state


Alternatives
------------

N/A


Policy
------

Using the Congress datalog syntax, write out an example policy using
https://wiki.openstack.org/wiki/Congress#Policy_Language

Example:

host_availability_zone(vm,zone_name) :-
    nova:hosts(host_name,zone=zone_name),
    nova:availability_zones(zone_name=zone_name)


Policy actions
--------------


Data sources
------------

nova


Data model impact
-----------------

N/A

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
  Joon Kang <joon-myung-kang>

Work items
----------

* Implement availability_zone_translator in nova datasource driver
* Perform unit testing
* Perform functional testing for the new field


Dependencies
============

N/A

Testing
=======

* Test host and availability zones using the policy

Documentation impact
====================

TBD

References
==========
[1] /congress/commit/9f9e26f850d12c73721b5e6b45ac00997c8c24c2
[2] http://docs.openstack.org/openstack-ops/content/scaling.html
