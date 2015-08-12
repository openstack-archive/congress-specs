..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
refactor-datasource-driver-framework
==========================================


https://blueprints.launchpad.net/congress/+spec/refactor-datasource-driver-framework

Currently the congress datasource driver is still pretty complex. This
blueprint aims to simplify it.


Problem description
===================

A detailed description of the problem:

* current there is a lot of logic in the convert_obj() method which is
  used to load the data into the data source driver. The majority of this
  logic is to validate that the table schema is valid. We should only need
  to do this one though on load of the data source driver.

* currently if you list the tables of a data source driver they aren't always
  populated in via __init__ self.state[table_name] = set(). I want to add a
  method to the DatasourceDriver base class that load_translator() which
  will allow one to pass in translators via the __init__ of a
  datasource driver. This will validate the schema there.

Proposed change
===============

Add load_translator() to datasource driver base class as descripted as able.
In addition to other refactoring that comes up as this work is being done.

Alternatives
------------

N/A

Policy
------

N/A

Using the Congress datalog syntax, write out an example policy using
https://wiki.openstack.org/wiki/Congress#Policy_Language

Example:

error(vm) :-
    nova:virtual_machine(vm),
    ids:ip_packet(src_ip, dst_ip),
    neutron:port(vm, src_ip), //finds out the port that has the VM’s IP
    ids:ip_blacklist(dst_ip).


Policy Actions
--------------

N/A
Describe the policy activities in terms of monitoring, reactive, proactive,
and other ways to explain how the policy will implement it's desired state.


Data Sources
------------

N/A
Describe which projects and/or services the data is coming from


Data model impact
-----------------

N/A
Changes which require modifications to the data model often have a wider impact
on the system.  The community often has strong opinions on how the data model
should be evolved, from both a functional and performance perspective. It is
therefore important to capture and gain agreement as early as possible on any
proposed changes to the data model.

Questions which need to be addressed by this section include:

* What new data objects and/or database schema changes is this going to
  require?

* What database migrations will accompany this change.

* How will the initial set of new data objects be generated, for example if you
  need to take into account existing instances, or modify other existing data
  describe how that will work.


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

Push code to do this.

Dependencies
============

N/A

Testing
=======

Unit tests and tempest tests will be present to confirm this works as desired.

Documentation impact
====================

Will update docs for how this now works.

References
==========

N/A
