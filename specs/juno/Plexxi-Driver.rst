..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Plexxi Driver
==========================================

https://blueprints.launchpad.net/congress/+spec/plexxi-driver

This blueprint is to layout the design of a driver that integrates PlexxiCore
into congress.

Problem description
===================

A detailed description of the problem:

    * A PlexxiCore Database can contain a vast wealth of information about a
      network that can be an asset to a Congress system in a network where
      Plexxi is present, and without this driver Congress does not have access
      to this data.


Proposed change
===============

Integrate the Plexxi Driver driver.


Alternatives
------------

None

Policy
------------

This example compares data stored in the Plexxi tables with data stored in
Nova tables and looks for Virtual Machines with names that are identical in
both data sets

RepeatedName(vname,pvuuid)
:- plexxi:vms(pvuuid,vname,phuuid,pvip,pvmaccount,pvaffin),
nova:servers(nvuuid,vname,a,nstatus,b,c,d,num)

Policy Actions
--------------

This driver provides monitoring of the Plexxi Core state as well
and currently demonstrations reactive actions based on the data provided.

Data Sources
------------

Primary: PlexxiCore
Secondary: Nova/Congress - For demo policy

Data model impact
-----------------

None

REST API impact
---------------

Users of Congress who have PlexxiCore integrated in their network will now be
able to access data stored in their PlexxiCore database through the congress
API.

Security impact
---------------

Network data stored in the PlexxiCore Database will now be accessible through
Congress

Notifications impact
--------------------

None

Other end user impact
---------------------

None

Performance impact
------------------

None

Other deployer impact
---------------------

None

Developer impact
----------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
    Conner-Ferguson

Work items
----------

None


Dependencies
============

Plexxi Class package
Plexxi-Core used within the network

Testing
=======


Documentation impact
====================

Documentation for the Driver is currently located in the bitbucket
that found that can be found in the references section

References
==========

Working Demo -
https://www.youtube.com/watch?v=ZAEydTlIW64
Current Code Location -
https://bitbucket.org/ConnerFerguson/plexxi-driver/overview
Devloper Email -
Conner.Ferguson1@marist.edu

