..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Removing d6cage dependency in api process
==========================================

https://blueprints.launchpad.net/congress/+spec/dist-remove-d6cage-from-api

remove d6cage dependency

Problem description
===================

API server currently relies on d6cage to create new API instances. Congress
will remove d6cage in the next distributed architecture. It needs to remove
dependency on d6cage of API instance creation.

Proposed change
===============

1. Introduce a new dict object to create instances of API models. The dict
   has references to each API instances d6cage has as services[name]['object']
   now. Then router.py:APIRouterV1 uses the references to register API
   with the instance.
2. Change base class of some instances, which aren't need to be a deepsix
   class. This BP also changes these kind of class to python normal object.
   SchemaModel and DatasourceModel aren't need to be deepSix's subclass both
   in the current architecture and in the distributed one. others aren't need
   to be deepSix's subclass in the new distributed one.

How to change to distributed version

It replace calling harness.py:create() in service.py:congress_app_factory()
with a new API instance creating method which has instantiation steps for
API model in harness.py:create(). And it's only to make the dict with new
instances.

Alternatives
------------

The alternative is to spread codes in harness.py:create() into
router.py:APIRouterV1.__init__(). This change is easier to track
in current one process solution.

How to change to distributed version

It need to replace all d6cage.createservice() in APIRouterV1.__init__()
with new API instance creating method. There are a lot of place we have
to change for it. So this way is an alternative for this BP.


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
  muroi-masahito

Other contributors:
  <launchpad-id or None>

Work items
----------

0. create a new dict to have references to API instances
1. pass it as an argument for router.py:APIRouterV1()
2. use it to register API and the instances
3. remove API instances which don't need to inherit deepsix from d6cage
4. change direct access with variable to indirect access with method

Dependencies
============

N/A

Testing
=======

- change Unit tests for non-deepsix object if needed

Documentation impact
====================

N/A

References
==========

N/A
