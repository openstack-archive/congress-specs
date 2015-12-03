..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Push Type DataSource Driver
==========================================

https://blueprints.launchpad.net/congress/+spec/push-type-datasource-driver

Problem description
===================

Currently DataSource Driver retrieves all cloud service's info by polling
each datasource. The purpose of the polling is just to monitor cloud
service periodically. Thus Congress can't notice a change in cloud service
just after the change occurs. It means a policy violation isn't detected
until Congress polls the change. Additionally, Congress calls the service's
API to get the data for a table, so datasource table's schema depends on API
response contents.

In the area of monitoring, tons of well-developed monitoring system have
already existed in the world. The systems have an ability to notify a change
in cloud services just after a change occurs. And then these monitoring systems
can detect many kind of changes in cloud services that Congress can't do.

Congress should retrive the notification as a DataSource since it helps
Congress to calculate a policy violation faster than now and also enables
us to define varied policies and rules.

Proposed change
===============

Following changes are proposed in the BP.

1. Create PollingDataSourceDriver and PushedDataSourceDriver class. Current

   DataSourceDriver has 2 features. One is converting data to datasource table
   and sending it to Policy Engine. Another is polling data from datasource.
   We should separate the polling feature into another class that is subclass
   of DataSourceDriver named PollingDataSourceDriver. And we create
   PushedDataSourceDriver, which is subclass of DataSourceDriver and receives
   data from a monitoring system.

2. Create HttpReceiverDriver subclass or OsloMsgReceiverDriver subclass

   Both Drivers are a subclass of PushedDataSourceDriver and receive
   datasource info by the protocol the class name has. DataSourceDriver has
   to keep datasource status in itself, so the classes will support 2 types
   of a data receiving method. One is receiving all status data each push
   time. Another is receiving only delta data that represents which row is
   deleted or added in the status like REST API.

   First of all, the BP chooses and implements one of them based on an usecase,
   which needs to use PushedDatasourceDriver.

There are some restrictions as a first implementation.

* Table definitions

  Ideally speaking, PushedDataSourceDriver should allow admin to change a
  datasource name, table name, schema and so on by Congress API.
  PushedDataSourceDriver defines the protocol name as a datasource name in
  first implementation. Then the admin can define a translator in another
  python file. In the current architecture, the path is set by 'config' key
  in a call for create datasource API. In the distributed architecture, the
  path is set by a config file for each datasource driver.

Alternatives
------------

The alternative is that Congress allows admin to add and delete a row by API
calls. The API is a REST API, so the benefit is that both create and delete
methods are already developed in itself. However, the way has following
bad points. First is all request needs an authentication to push data. It
means all datasource have to be related to keystone authority. Second is
it's not easy to divide a network segmentation based on each API's purpose.
If Congress enables API to retrieve datasource, the datasource receiver is
exposed to user.


Policy
------
Example: define a table that shows service name related to  host's failures
         detected by monitoring system.

Assumption: monitoring system pushes data by http. col_1 represents name of
            a host where the monitoring system detects
            some failure. col_2 represents a failure type.

host_failures(host):-
  http:table(col_1=host, col_2="nic_error")

host_failures(host):-
  http:table(col_1=host, col_2="disk_error")

failure_effected_service(service, host) :-
    nova:service(host_name=host, service=service),
    host_failures(host=host)

Policy actions
--------------

N/A

Data sources
------------

Add new datasource model and table.


Data model impact
-----------------

N/A

REST API impact
---------------

Creating new datasource name, table name and table schema by API is a future
work. It means this feature is out of scope for this BP.

Security impact
---------------

It enables others to push datasource row into the datasource driver. It would
cause undesired policy violation if malicious users send data on purpose.

Notifications impact
--------------------

N/A

Other end user impact
---------------------

N/A

Performance impact
------------------

PushedDataSourceDriver notifies a status change when it receives a change from
a monitoring system. If the system frequently sends a notification, it would
make Congress busy with calculating policy table.

Other deployer impact
---------------------

* Add a config for column number
* Disable the PushedDataSourceDriver in default because of a reason described
  in Security Impact section

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
  None

Work items
----------

0. Create PollingDataSourceDriver subclass and pull out methods related to
   polling from DataSourceDriver class into the new Driver
1. Create PushedDataSourceDriver subclass
2. Create HttpReceiverDriver or OsloMsgReceiverDriver

Dependencies
============

N/A

Testing
=======

* Add unit tests related to new class
* Add tempest scenario tests which use pushed data in rules if possible

Documentation impact
====================

DataSource list will be updated if the list is described in docs.

References
==========

Tokyo Summit discussion: https://etherpad.openstack.org/p/congress-mitaka-external