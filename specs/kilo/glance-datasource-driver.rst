..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Glance Data Source Driver
==========================================

https://blueprints.launchpad.net/congress/+spec/glance-data-source-driver

This blueprint is to add a data source driver for glance.

Problem description
===================

N/A

Proposed change
===============

Add data source driver that integrates congress with glance.

Alternatives
------------

N/A

Policy
------

This will use the congress language. glance:image(X) etc.

Policy Actions
--------------

Just monitoring right now.

Data Sources
------------

Glance

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

Primary assignee:
    arosen

Work items
----------


Dependencies
============

python-glanceclient

Testing
=======

Will add tempest tests.

Documentation impact
====================

N/A

References
==========

N/A
