..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============================
Create a Heat Datasource Driver
===============================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/congress/+spec/heat-datasource-driver

This Heat driver will allow Congress to interact with the Openstack Heat API
for orchestration. The first version will provide data from Heat's read API
calls until Congress does has infrastructure to handle writing to drivers.
Subsequent versions may be able to send requests and write to the Heat API.


Problem description
===================

Today, there is no Congress driver for Heat, either for reading or writing.
This driver will give Congress eyes into Heat so that a policy writer can
inspect Heat state such as details about each stack, stack snapshots,
resources, software configurations, software deployments, and perhaps
templates themselves.


Proposed change
===============

This driver will be similar to other existing drivers like neutron_driver.py
and nova_driver.py.  The Heat driver will read data from the following Heat
API calls and convert the responses to Congress tables:
* list stacks
* show stack detail
* list snapshots
* list resources
* show resource data
* show resource metadata
* list resource types
* show configuration details
* list stack events
* show event details
* show software_configs
* show software_deployments

Alternatives
------------

The alternative is to have no driver for Heat, which is not a good option for
those admins that use Heat in their cloud.


Policy
------

N/A

Policy actions
--------------

N/A

Data sources
------------

Openstack Heat

Data model impact
-----------------

N/A

REST API impact
---------------

N/A

Security impact
---------------

The Heat driver will need to authenticate to the Heat API just like all the
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
the impact of the API calls on Heat and Congress performance.

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
   tengqim

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
still dependent on the Heat API, so the tempest test is particularly important
as an integration test.


Documentation impact
====================

N/A


References
==========

Blueprint:
  https://blueprints.launchpad.net/congress/+spec/heat-datasource-driver
