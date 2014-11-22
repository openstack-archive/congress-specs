..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
persistent-storage-for-api
==========================================

https://blueprints.launchpad.net/congress/+spec/persistent-storage-for-api


Problem description
===================

* Currently we store all the api configuration data in memory so when the
  server is restarted all of ones data is lost. This blueprint is to
  implement a persistent layer to save this info.

* Needs to handle schema changes and data migration


Proposed change
===============

Add Persistance layer

Alternatives
------------

Use a flat file which might be easier for the first implementation but is more
error prone than using a db and doesn't let us scale out horizontally easily.

Policy
------

N/A

Policy actions
--------------

N/A


Data sources
------------

N/A

Data model impact
-----------------

N/A

REST API impact
---------------

N/A

Security impact
---------------

There could be some private data here as we are storing peoples policies here.
Though access to the db will require one to authenticate so hopefully there
is no security flaws here that allow uninteneded access.


Notifications impact
--------------------

N/A

Other end user impact
---------------------


*  Users will now have to deploy a db for congress to leverage.

Performance impact
------------------

* Hopefully none

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

Implementation

Dependencies
============

* We'll leverage alembic to handle the migrations and reuse a lot of the code
  that neutron already has to do this.

Testing
=======

Unit tests will be provided

Documentation impact
====================
N/A

References
==========

http://alembic.readthedocs.org/en/latest/
https://github.com/openstack/neutron/blob/master/neutron/db/migration/README

