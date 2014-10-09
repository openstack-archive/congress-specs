..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================================
Convert the DSE from using threads to using eventlets
=====================================================

https://blueprints.launchpad.net/congress/+spec/dse

Currently, the OpenStack standards are to use eventlets over threads.  The
DSE framework was merged into Congress using threads and needs to be
changed to use eventlets to be more in line with the OpenStack standards.

Problem description
===================

The DSE currently uses threads for each of the services it loads, while
the rest of OpenStack uses eventlets over threads.  Eventlets can scale
better than threads and are able to be stopped, while threads cannot be
stopped with a single method.  This raises a problem because if there
is a limit to the amount of threads that can be run without causing
performance issues.

Proposed change
===============

The change that I propose is to make the deepSix class inherit from
green thread.GreenThread (from the eventlet python module) rather than inheriting
from threading.Thread.  This will make each service a greenthread rather than
a thread.

Also, when the DSE creates the services in the create service method, it
will spawn them as greenthreads, rather than start threads.

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

The loading and starting of a d6service will change.

Old way (taken from test_dse.py lines 31-35):
    cage = congress.dse.d6cage.d6cage()
    # so that we exit once test finishes; all other threads are forced
    #    to be daemons
    cage.setDaemon(True)
    cage.start()

New way:
    cage = deepsix.spawnService(congress.dse.d6cage.d6Cage)
    cage.d6stop() # Stops the cage and kills the greenthreads

Performance Impact
------------------

This code will be called every time a d6service object is created. The
implemented code should not have any effects on performance since it
is executed as quickly as threading is.

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
  harrison.kelly (email: harrison.kelly@plexxi.com)

Work Items
----------

Change d6cage.py to run each service as a greenthread instead of a thread.

Change deepsix.py to inherit from greenthread.GreenThread instead of
threading.Thread and spawn greenthreads instead of threads.


Dependencies
============

None


Testing
=======

Tests with DSE modules will be used to check if implementation does not effect
the capabilities of the DSE.


Documentation Impact
====================

None


References
==========

None
