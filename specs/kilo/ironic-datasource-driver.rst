..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================================
Create an ironic Datasource Driver
==================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/congress/+spec/ironic-datasource-driver

This ironic driver will allow Congress to interact with the Openstack ironic
API for orchestration. The first version will provide data from ironic's read
API calls until Congress does have infrastructure to handle writing to drivers.
Subsequent versions may be able to send requests and write to the ironic API.


Problem description
===================

Today, there is no Congress driver for ironic, either for reading or writing.
This driver will give Congress eyes into ironic so that a policy writer can
inspect Bare Metal state such as details about enabled drivers, each chassis,
each node, each port, supported boot devices of a node.


Proposed change
===============

This driver will be similar to other existing drivers like neutron_driver.py
and nova_driver.py.  The ironic driver will read data from the following ironic
API calls and convert the responses to Congress tables:

* list enabled drivers
* show driver detail
* list driver properties
* list chassis
* show chassis detail
* list nodes contained in a chassis
* list registered nodes
* show node detail
* list supported boot devices for a node
* list ports assocaited with a node
* list all registered ports
* show port detail

Alternatives
------------

The alternative is to have no driver for ironic, which is not a good option for
those admins that use ironic in their cloud to manage bare metal servers.


Policy
------

N/A

Policy actions
--------------

N/A

Data sources
------------

Openstack ironic

Data model impact
-----------------

N/A

REST API impact
---------------

N/A

Security impact
---------------

The ironic driver will need to authenticate to the ironic API just like all the
other datasource drivers.

Notifications impact
--------------------

N/A

Other end user impact
---------------------

N/A

Performance impact
------------------

It is possible that the API calls will be expensive.  We will need to measure
the impact of the API calls on ironic and Congress performance.

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
   zhenzan-zhou

Work items
----------

 * Write unit test
 * Write tempest test
 * Write API client code
 * Write translators for tables


Dependencies
============

N/A


Testing
=======

This work must include a unit test and a tempest test.  The driver translator
infrastructure makes most of the translation code robust, but the driver is
still dependent on the ironic API, so the tempest test is particularly
important as an integration test.


Documentation impact
====================

N/A


References
==========

Blueprint:
  https://blueprints.launchpad.net/congress/+spec/ironic-datasource-driver
