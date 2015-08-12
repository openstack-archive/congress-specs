..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
murano-congress-integration
==========================================

https://blueprints.launchpad.net/congress/+spec/murano-congress-integration

This blueprint is to convert the integration with murano and congress.

Problem description
===================

A detailed description of the problem:

* Make murano and congress interface with each other.

Proposed change
===============

This blueprint covers 2 different things that we need to do.

- The first is write a datasource driver for congress that interacts with
  murano.

- The second is to integrate murano with congress. To do this we might want
  to provide some kind of congressmiddeware if there are any details we can
  abstract there but we will figure this out later after that datasource
  driver is written.

Alternatives
------------

N/A

Policy
------

N/A

Policy Actions
--------------

N/A

Data Sources
------------

The Murano Project introduces an application catalog to OpenStack,
enabling application developers and cloud administrators to publish
various cloud-ready applications in a browsable categorized catalog.
Cloud users -- including inexperienced ones -- can then use the catalog
to compose reliable application environments with the push of a button [1]

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
    Murano Team

Work items
----------

Data source driver
murano integration with congress api

Dependencies
============

Data source driver for murano should be done first.

Testing
=======

Tempest and unit tests will be added.

Documentation impact
====================

N/A

References
==========

[1] - https://wiki.openstack.org/wiki/Murano
