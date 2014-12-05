..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
vCenter Driver
==========================================

https://blueprints.launchpad.net/congress/+spec/vcenter-driver

This blueprint is to add a data source driver for vCenter, giving congress
access to  new information from an external datasource.

Problem description
===================

N/A


Proposed change
===============

Add a data source driver that integrates congress with vCenter by connecting
to vCenter using oslo.vmware.


Alternatives
------------

N/A


Policy
------

This will use the congress language. vCenter:hosts(X) etc.

Example - Creating a whitelist of all MAC addresses of hosts found in vCenter

WhiteList(vnic_macs,pnic_macs) :-
    vCenter:hosts(host:vnic_mac_id,host:pnic_mac_id),
    vCenter:host.pnic_macs(host:pnic_mac_id,pnic_macs),
    vCenter:host.vnic_macs(host:vnic_mac_id,vnic_macs)

Policy Actions
--------------

Monitoring Hosts and Virtual Machines


Data Sources
------------

vCenter


Data model impact
-----------------

N/A

REST API impact
---------------

N/A

Security impact
---------------

This driver will require vCenter credentials to be input into congresses
configuration, and will provide congress data by using those credentials. It
will be important for those implementing this driver to be aware of what data
is visible from congresses API.

Notifications impact
--------------------

N/A

Other end user impact
---------------------

N/A

Performance Impact
------------------

Implementing this driver will add another data source for congress to parse
data from, and since this driver pulls data from a non-openstack source this
will generate additional traffic on the network.

Other Deployer Impacts
----------------------

To use this driver a deployer will need to configure this driver in
datasource.config.

Developer Impact
----------------

N/A


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Conner Ferguson


Work Items
----------

N/A


Dependencies
============

N/A


Testing
=======

TBD


Documentation Impact
====================

Documentation can already be found at
https://bitbucket.org/ConnerFerguson/vcenter-driver


References
==========

https://bitbucket.org/ConnerFerguson/vcenter-driver - Current code hosting


