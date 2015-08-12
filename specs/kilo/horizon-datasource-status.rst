..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================================
Congress OpenStack Horizon : Data Source Status Table
=====================================================


https://blueprints.launchpad.net/congress/+spec/horizon-datasource-status-table

This blue print describes integration of datasources status with Horizon
dashboard. Admin will be able to see detailed status of each datasource.

Problem description
===================

A detailed description of the problem:

* In the existing horizon implementation congress data source status is not
  shown

* This poses problem to admins who want to see the status of data sources


Proposed change
===============

* Proposal is to add data source status table to the landing page
    contrib/horizon/datasources/templates/datasources/index.html
* Add appropriate table to the file /contrib/horizon/datasources/tables.py

Alternatives
------------

none


Screens
-------

none


Policy actions
--------------

none


Data sources
------------

none


Data model impact
-----------------

none


REST API impact
---------------
 no updates required


Security impact
---------------

Authentication of python-congressclient through Keystone token

Notifications impact
--------------------

none

Other end user impact
---------------------

* User will be able to view Data Source Status



Performance impact
------------------

none

Other deployer impact
---------------------

none

Developer impact
----------------

none


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <jwy>
  <rajdeepd>

Other contributors:
  <None>

Work items
----------

* Modify base Panel for Policies and Data sources tables
* Add congress.py to add calls for getting status information
* Create a Data Source status table

Dependencies
============

* Horizon
* python-congressclient


Testing
=======

Unit testing using mocks.


Documentation impact
====================

none


References
==========

https://github.com/stackforge/python-congressclient
