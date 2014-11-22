..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================================================
Congress OpenStack Horizon Dashboard Integration
================================================


https://blueprints.launchpad.net/congress/+spec/horizon-integration

This blue print describes integration of Congress with Horizon dashboard.
Admin should be able to create/update or view policies and rules. He should
also be able to see information fetched from the various data sources.


* ..
*

Problem description
===================

A detailed description of the problem:

* In the existing implementation congress data elements like policies, rules
  and data sources can be accessed only through command line or python client.

* This poses problem to admins who want to use UI based dashboard to manage
  policies and its associated elements



Proposed change
===============

* Proposal is to integrate Congress read/ write and update use cases into
  Horizon
* A new panel named `policies` will be added to the `admin` dashboard.

+-----------+                          +------------+
|           ++------+                  |            |
|           ||      |     REST API Call|            |
|           ||      +------------------>            |
| OpenStack ||  congress               |  Congress  |
|  Horizon  ||  python-client          |  API       |
|           ||      |                  |  Service   |
|           ||      |                  |            |
|           ||      |                  |            |
|           ||      |                  |            |
|           ||      |                  |            |
|           ++------+                  |            |
+-----------+                          +------------+

* This Panel will have a Tab Group

    * First Tab will be Policies Tab
    * Second Tab will be DataSources Tab

    Policies Tab will cover following use cases

    * List of policies
    * List of Rules in a Policy
    * List of tables in a Policy
    * Create a Rule
    * Update a Rule
    * Delete a Rule

    DataSource Tab will show

    * List of DataSources
    * Tables returned by DataSources

Alternatives
------------

Implement a dashboard which is independent of Horizon, in case there is a need
to integrate congress in a non openstack scenario.


Screens
-------
 none


Policy Actions
--------------

none


Data Sources
------------

none


Data model impact
-----------------

none


REST API impact
---------------

To be determined.
We might need some additional data to be exposed by python-congressclient


Security impact
---------------

Authentication of python-congressclient through Keystone token

Notifications impact
--------------------

none

Other end user impact
---------------------

* User will be able to view, configure and update Policies, Rules.
* User will be able to data exposed by the DataSources


Performance impact
------------------

none

Other deployer impact
---------------------

Integration with Devstack.

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

* Add base Panel for Policies
* TabGroup for Policies and DataSources
* Implement Policy Tab
* Implement DataSource Tab


Dependencies
============

* Horizon

* python-congressclient



Testing
=======

Unit testing using mocks.


Documentation impact
====================

Document the screenflow.


References
==========

https://github.com/stackforge/python-congressclient
