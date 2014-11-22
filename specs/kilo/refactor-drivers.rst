..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============================================================================
Pull common functionality into driver superclass, including data transformation
===============================================================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/congress/+spec/refactor-drivers

Currently, each datasource driver contains code to convert data from the API
call response to the Congress data tables.  This change will make it easier
for a developer to add each incremental data source driver.


Problem description
===================

Today, it takes more code than necessary to write a data source driver.

The code is also more difficult to write than necessary. Some of these API
responses (like in neutron list networks) contain nested data, for example in
the list of networks, each network can contain a sub-list of subnets. The
driver populates a separate table for subnets and the creates a key to link
between the network table and the subnet table.


Proposed change
===============

To generalize the driver, this change will allow a driver class to specify how
to extract data from the API response, and into which table/field to put the
response data.

To handle the sublist conversion, the driver class will specify if a field is
a sublist, and then also specify a key, so that the main table can link to the
subtable.  This sublist relationship will be recursive, so that a sublist can
also contain another sublist.


Alternatives
------------

None


Policy
------

None


Policy Actions
--------------

None


Data Sources
------------

None


Data model impact
-----------------

None


REST API impact
---------------

None


Security impact
---------------

None


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

This change will have an effect as soon as it is merged, but Congress should
behave exactly as it did without the change.


Developer impact
----------------

The new refactoring will change how programmers write datasource drivers.  The
new way should be easier, require less code, and be less bug prone.  To take
advantage of the new refactoring, we'll need to rewrite the existing drivers,
but this need not happen at the same time as writing the new datasource driver
superclass.


Implementation
==============

Assignee(s)
-----------

ayip


Work items
----------

1) Write new superclass.
2) Rewrite Nova driver.
3) Rewrite Neutron driver.
4) Rewrite Keystone driver.


Dependencies
============

None


Testing
=======

Unit tests for superclass, and modify existing test cases for individual
drivers.


Documentation impact
====================

This change will include new documentation for how to use the new datasource
driver superclass.


References
==========

None
