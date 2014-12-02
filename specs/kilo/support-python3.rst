..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================================
Add Support for Python 3 to Congress
====================================

https://blueprints.launchpad.net/congress/+spec/support-python3

This specification describes how to gradually add Python 3 support to
Congress.


Problem description
===================

Currently Python 3 tests are failing. In an effort to support both Python 2
and Python 3 concurrently a number of changes to Congress are needed.


Proposed change
===============

This specification will be used to track a number of successive, minor changes
to Congress with the goal of fully supporting Python 3. Each commit will address
one change to Congress to ensure compatibility with Python 3 while continuing
support for Python 2.


Alternatives
------------

None.


Policy
------

None.


Policy Actions
--------------

None.


Data Sources
------------

None.


Data model impact
-----------------

None.


REST API impact
---------------

None.


Security impact
---------------

None.

Notifications impact
--------------------

None.

Other end user impact
---------------------

None.

Performance Impact
------------------

None.

Other Deployer Impacts
----------------------

None.

Developer Impact
----------------

Once fully implemented commits must pass tox tests against py34 and should be
rejected if tests fail.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  jzabala

Other contributors:
  None

Work Items
----------

- Determine what sections of code need to be modified to ensure Python 3
  compatibility. This can be done by generating list of fixers which run
  when '2to3' is used to transform code to Python 3.
- Run code through 2to3 for each fixer (see previous bullet), also ensuring that
  changes to the code do not break compatibility with Python 2.
- Confirm that changes are gradually improving tox -e py34 test outcomes.



Dependencies
============

tbd


Testing
=======

Implementation of this specification should result in tox tests against py34
succeeding. Current unit tests (and the code being tested) may have to be
modified during the course of the implementation of this specification.


Documentation Impact
====================

None.


References
==========

None.
