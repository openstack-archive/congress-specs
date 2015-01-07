..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Parent key improvements
==========================================

https://blueprints.launchpad.net/congress/+spec/parent-key-improvements

There are two current shortcomings of the parent_key implementation in
congress.

* It's unable to be accessed from a subtable. This is needed in the neutron
  refactoring working to handle how neutron structures the response for
  routers.

* It's column name is always named parent_key. It would be helpful if we
  could have the column in the schema actually reflect it's name.
  For example, router_id per say.


Problem description
===================

* Parent key cannot be accessed from subtables which is needed.
* No way to rename parent key column name.

Proposed change
===============

Change the datasource framework code so this can be done.

To handle specifiy the column name on a parent_key on should add:
'parent-col-name': <NAME> to the translator. If this is not present
then the old behavior will still occur of having the column name called
parent_key.

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

N/A

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
   arosen

Work items
----------

Implement

Dependencies
============

* This needs to be done before neutronv2 refactoring can be done.


Testing
=======

Unit tests will be included.

Documentation impact
====================

N/A

References
==========

N/A
