..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Swift Data Source Driver
==========================================

https://blueprints.launchpad.net/congress/+spec/swift-data-source-driver

This blueprint is to add a data source driver for swift.

Problem description
===================

A datasource driver is required to expose list of containers and
list of objects in each container to the Congress policy framework
so that we can write policies involving object storage entities.
Swift is the OpenStack component that provides object storage service.
The swift-datasource-driver interacts with swift service to provide
object storage specific states to congress for policy monitoring.

Proposed change
===============

Add data source driver that integrates congress with swift.

Alternatives
------------

N/A

Policy
------

This will use the congress language. swift:containers(X) etc.

Policy actions
--------------

Just monitoring right now.

Data sources
------------

Swift

Data model impact
-----------------

TBD

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

Srinivasa Rao Ragolu
sragolu@mvista.com

Work items
----------

- Implement swift driver with essential tables
- Implement test code to test the driver
- Implement tempest code for real-time tests

Dependencies
============

python-swiftclient

Testing
=======

- Will need to add unit test code
- Will add tempest tests.

Documentation impact
====================

N/A

References
==========

N/A
