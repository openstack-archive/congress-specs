..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Add Extract-fn To All Translators
==========================================

https://blueprints.launchpad.net/congress/+spec/add-extract-fn-to-all-translators

This proposal would expand the general functionality of the translate_obj()
method by adding a general extract-fn method to translators for HDICT, VDICT,
and LIST. This gives datasource developers versatile tool to use when
translating data into congress congress drivers.


Problem description
===================

Today, if a datasource returns an object that contains a json string instead
of pure python objects, lists, and dicts, then the translator has no way to
read data out of the json object.

Proposed change
===============

By including the extract-fn method as a translator, the translate_obj() method
could then preprocess the data object with extract-fn before running the
existing translation code. This way,the programmer can set the extract-fn to
a json extractor to convert the json into a python object.


Alternatives
------------

Currently if a datasource returns data in a non-standard format a driver
developer can create unique methods inside their drivers to handle this data.
This proposed change potentially eliminates the need for developers to
individually generate code to use in their solutions by providing a tool to
assist with translations in base driver class.


Policy
------

None


Policy actions
--------------

None


Data sources
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

None

Developer impact
----------------

This change provides developers more versatility when creating datasource
drivers.


Implementation
==============

Assignee(s)
-----------


Primary assignee:
  Alex Yip

Other contributors:
  Conner Ferguson


Work items
----------

Add extract-fn as a parameter to HDICT, VDICT, and LIST translators.


Dependencies
============

None


Testing
=======

To test the results of this change, data will need to be passed through the
extract-fn translator then this data will be checked to ensure that it is
being displayed in the expected manor.


Documentation impact
====================

Minor additions to the datasource driver based class documentation to inform
new developers about what the extract-fn method does as a translator.


References
==========

None
