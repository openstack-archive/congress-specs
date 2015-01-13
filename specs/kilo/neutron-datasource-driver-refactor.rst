..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
neutron-datasource-driver-refactor
==========================================

https://blueprints.launchpad.net/congress/+spec/neutron-datasource-driver-refactor

This blueprint is to cover the refactoring work to improve how the neutron
datasource driver structures its data. In addition, to add security-group-rule
support.

Problem description
===================

* Refractor neutron driver table schema.
* Add security group/security-group-rule support

Proposed change
===============

* Leverage parent-key attribute to remove subtables.
* Restructure table schema to be more natural.
* Implement security-group/security-group-rule support.

Alternatives
------------

N/A

Policy
------

N/A

Policy actions
--------------

N/A

Data sources
------------

Neutron


Data model impact
-----------------

The schema will change to remove the subtables. This will simplify how one
writes policy though it will break existing policies that have been written
against the neutron driver.

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

The datasources.conf file will have to be updated to include the new
neutron driver.

Developer impact
----------------

N/A

Implementation
==============

Assignee(s)
-----------

Who is leading the writing of the code? Or is this a blueprint where you're
throwing it out there to see who picks it up?

If more than one person is working on the implementation, please designate the
primary author and contact.

Primary assignee:
	arosen

Work items
----------

Implement

Dependencies
============

All other patches this requires have already been merged into congress.

Testing
=======

Tempest and unittests will be present.

Documentation impact
====================

Will update docs

References
==========

N/A
