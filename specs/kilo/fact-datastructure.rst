..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================
Create a Fact class for storing facts
=====================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/congress/+spec/fact-datastructure

Today, the congress runtime stores facts as Rules. This is inefficient from a
memory perspective since each Rule contains a Literal, which in turn contains
a Term for each column. Each of these is a python object, and each of them
also contains a few extra fields like head, body, location, negated,
etc. Using Rules is also inefficient with CPU since congress needs to
construct all these objects. This blueprint proposes to use a Fact data
structure to store each fact. A Fact is a subclass of a native tuple, plus one
field for table name. This is much more efficient memorywise and CPUwise than
using a Rule because there are no extra objects like Literal and
Term. Preliminary tests show a 10x reduction in CPU for initializing tables
plus a 3x reduction in memory use.


Problem description
===================

A detailed description of the problem:

* Today, congress stores each fact as a Rule object

* A Rule object contains many objects and fields

* Many objects and fields means that creating and storing a fact uses lots of
  CPU and memory resources.

* High CPU and memory use makes congress unable to scale to larger datasets.


Proposed change
===============

We propose to create a new class called Fact to store each fact.  A Fact is a
native tuple plus one string for table name.  Using a Fact will eliminate all
the subfields and subobjects in Rule.


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

Preliminary testing shows a 10x reduction in CPU use and a 3x reduction in
memory use in initialize_table() for 7M facts where the payload is 700MB.

Other deployer impact
---------------------
None

Developer impact
----------------

A Theory object will internally contain Rules and Facts.  The caller an insert
a Fact into a RuleSet.  However, whenever someone fetches the rules from a
Theory the RuleSet converts Facts to Rules before returning them.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  ayip

Work items
----------

* Implement FactSet

* Use FactSet inside of RuleSet

* Change initialize_tables() to avoid instantiating a list of all facts coming
  from DSE

Dependencies
============

None

Testing
=======

Add a unit test for FactSet

Documentation impact
====================

None

References
==========

None
