..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============================================
Public-Private Networks with Group Membership
=============================================

https://blueprints.launchpad.net/congress/+spec/public-private-networks-with-group-membership

Problem description
===================

There are a plethora of restrictions that cloud operators may want to place on
how VMs are connected to networks. For example, suppose we want to ensure that
every network a VM is connected to is either a public network or the owner of
the network and the other of the VM belong to the same
ActiveDirectory/Keystone/etc. group.


Proposed change
===============



Alternatives
------------


Policy
------

error(vm) :-
nova:instance(vm),
nova:network(vm, network),
not neutron:public(network),
nova:owner(vm, vmowner),
neutron:owner(network, netowner),
not same_group(vmowner, netowner)
same_group(x,y) :-
	group(x,g),
	group(y,g)
group(x,g) :-
	keystone:group(x, g)
group(x,g) :-
	ad:group(x,g)


Policy Actions
--------------

* Monitoring  (Could also illustrate proactive/reactive actions).


Data model impact
-----------------

* Neutron: the list of public networks and the owner of each network
* Nova: the list of networks connected to VMs and the owners of those VMs

ActiveDirectory/Keystone: which users are members of which groups


REST API impact
---------------



Security impact
---------------



Notifications impact
--------------------



Other end user impact
---------------------


Performance Impact
------------------



Other deployer impact
---------------------



Developer impact
----------------



Implementation
==============


Assignee(s)
-----------


Work Items
----------



Dependencies
============



Testing
=======



Documentation Impact
====================



References
==========


