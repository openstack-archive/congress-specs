..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================
Selectability for Translator
============================

https://blueprints.launchpad.net/congress/+spec/selectable-translator

Problem description
===================

DataSource Driver creates all translator and publishes its schema when
it's initialized. If the driver is one of polling driver, it polls data
of all translator from related services, even though any policy rules don't
use the schema. The behavior causes performance issues when an unnecessary
translator takes a long time to fetch data from related service.

Admin can choose which DataSource Driver they use or don't use by configuring
congress.conf and creating DataSource by API. On the other hands, there is
no way for admin to choose which translator in the created DataSource publishes
their schema. So when the update frequency of an unnecessary translator is
high, the issue described above also happens.

Proposed change
===============

This BP proposes a new feature that admin can set *lazy* to polling
data from services until other service requires schema of the datasource.
The issue described above doesn't happen in the usual case. So datasources
poll data for all table in default. If admin don't want a datasource to
poll unnecessary data, they disableds polling flag of the datasource and then
the datasource doesn't poll data if not needed.

Admin defines a list of lazy tables in datasource config parameter when
he/she instantiates a new datasouce. The datasource doesn't poll data related
to tables listed in the config value while any policy rule doesn't subscribe
the table. If datasource notices/recieves the subscription it starts to poll
data.

There are 2 possible corner cases.

* lazy parent translator and non-lazy child translator

  This issue happens when parent translator is defined as a lazy but child
  translator is not defined. Each table is mapped to one translator and
  some translators have a hierarchy for extracting one pulled data to some
  tables. When admin specifies a table mapped to a child translator as a lazy
  but parent translator is not lazy, details of the issue are that the
  datasource should pull data or not.

  In this proposal, the datasource tries to pull data. Having the hierarchy
  is depending on an implementation of the client the datasource uses. If
  the client has separeted methods for pulling both data the datasource could
  use each method.

* list table row API for a lazy table

  This issue happens when user calls list table row API for a lazy table before
  any service subscribes it. The table has no data about the table and the
  datasource can't return row data. Details of the issue are that the
  datasource should pull data and return a response of row or not when user
  calls the API.

  In this proposal, the API returns 400 Bad Request error with a message about
  the table doesn't start to poll since the table is defined as lazy. If the
  API returns a response with row data user can't recognize the table is a lazy
  and the row data is up-to-date or not.

Alternatives
------------

Specifying a table that doesn't poll is one of alternatives. This allows
admin to handle each table to poll or not. Additionally, the datasource
doesn't pull data in not polling status even though policy rules using the
table are defined. But it also requires admin to manage all table manually.
It's hard to operate.

Enabling admin to config polling time delta for each translator is one of
answers to resolve a polling performance issue. The good point is admin
can write a policy rule related to the translator anytime since PolicyEngine
has its schema. The down side, however, is policy calculation could be delayed
since the delta is longer than in usual case.

Admin can set the enabled flag by config file for each translator too.
Using config file is an easy way to disable it and has low impacts for codes.
However, he/she have to restart DataSource everytime they want to change the
flag.

Policy
------

N/A

Policy actions
--------------

N/A

Data sources
------------

Datasource Driver have a functionality to start/stop pull data from service.
Datasource starts to pull ALL data in default just after initialized. If lazy
config is specified to some tables, the datasource waits to start to pull data
related to the tables until other service subscribes the table. And then when
there is no subscriber for the lazy table the datasource stops to pull data.

There is no change from PolicyEngine perspectives.

Data model impact
-----------------

Datasources have a list of lazy table in its config values. Currently, all
config values are stored in DB, so the list is also stored there.

REST API impact
---------------

This proposal expands parameters of existing APIs.

* Define a list of lazy tables

  * API Schema

    POST /v1/data-sources

  * Request Body

   .. code-block:: javascript

      {
          "config": {
              ...
              "lazy_table": "[table1-a, table-b]",
              ...
          },
          ...
      }

  * Response Body

   .. code-block:: javascript

      {
          "config": {
              ...
              "lazy_table": "[table1-a, table-b]",
              ...
          },
          ...
      }

List show a datasource details

  * API Schema

    GET /v1/data-sources/<datasoure-id>

  * Response Body

   .. code-block:: javascript

      {
          "config": {
              ...
              "lazy_table": "[table1-a, table-b]",
              ...
          },
          ...
      }

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

As described in Problem description section, the feature is to improve
PollingDataSource Driver. It reduces polling time and amount of sent data
from DataSource to PolicyEngine.

Other deployer impact
---------------------

N/A

Developer impact
----------------

Developers who write another PollingDataSource Driver

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  muroi-masahito

Other contributors:
  muroi-masahito

Work items
----------

0. Disable polling data
   When no subscrier exists for lazy tables, PollingDataSourceDriver
   stops pulling data. And the driver starts pulling data after some
   other service start to subscribe the table.
1. "lazy_table" config
   Subclass of PollingDataSourceDriver will have "lazy_table" key in
   its config parameter. This key is optional config.

Dependencies
============

N/A

Testing
=======

* Add unit tests related to the functionality

Documentation impact
====================

API reference will be updated based on the API changes.

References
==========

N/A