..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Murano Data Source Driver
==========================================

https://blueprints.launchpad.net/congress/+spec/murano-driver

This blueprint is to add a data source driver for Murano

Problem description
===================

We need a data source to get the Murano environments and applications related
data so that we can define policies during and after application service
fulfillment

Proposed change
===============

Add data source driver that integrates Congress with Murano with following
data tables.

* Data Tables

   * *murano:objects(object_id, owner_id, type)*

      This table holds every MuranoPL object instance in an environment.

        * *object_id* - uuid of the object as used in Murano
        * *owner_id* - uuid of the owner object as used in Murano
        * *type* - string with full type identification as used in Murano (e.g., io.murano.Environment,...)

   * *murano:parent-types(id, parent_type)*

      This table holds parent types of *obj_id* object.

        * *id* - uuid of the object as used in Murano
        * *parent_type* - string with full type identification of the parent object

      Note that Murano supports multiple inheritance, so there can be several
      parent types for one object

   * *murano:properties(id, name, value)*

      This table stores object's properties. For multiple cardinality
      properties, there can be number of records here.

        * *id* - uuid of the object as used in Murano
        * *name* - property name
        * *value* - property value

      MuranoPL properties referencing class type (i.e., another object)
      are stored in *murano:relationship*. Properties with structure has
      not direct mapping here - it has to be stored here component
      by component.

   * *murano:relationships(source_id, target_id, name)*

      This table stores relationship between objects (i.e., MuranoPL property
      to *class*).

        * *source_id* - uuid of the source object
        * *target_id* - uuid of the target object
        * *name* - name of the relationship between source and target object

      For multiple cardinality relationships there is several records in the
      table.

   * *murano:states(id, state)*

      This table stores *EnvironmentStatus* of Murano environment

        * *id* - uuid of the environment
        * *state* - the state status of the environment

      The state status will be one of 'READY', 'PENDING', 'DEPLOYING',
      'DEPLOY_FAILURE', 'DELETING', 'DELETE_FAILURE'.

Alternatives
------------

N/A

Policy
------

We will have pre and post deployment policies for Murano applications based
on the data tables defined in the driver.

Pre deployment policies will cover policies during application service
fulfillment and will leverage the Congress simulate feature.

Post deployment policies will cover monitoring policies on fulfilled service
instances.

Policy actions
--------------

Actions will be added in the second phase.

Data sources
------------

Murano

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

kishan-thomas
kishan.thomas@hp.com

Work items
----------

- Implement driver and data tables
- Implement unit tests
- Integration tests with Murano
- Add example policies

Dependencies
============

Murano API

Testing
=======

- Unit tests
- Integration tests
- Tempest tests

Documentation impact
====================

Add new documents in standard documentation location


References
==========

https://etherpad.openstack.org/p/policy-congress-murano-spec
