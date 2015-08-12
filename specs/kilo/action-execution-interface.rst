..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Add action execution interface
==========================================

https://blueprints.launchpad.net/congress/+spec/action-execution-interface

Add analogy to datasource driver for services that can execute actions.


Problem description
===================

Datasource drivers pull information from cloud services, allowing Congress
to read the current state of the cloud (e.g. the list of all servers).
Some cloud services also have the capacity to change the state of the cloud
(e.g. create a new server).  Today there is no way for the policy engine
to ask for changes to be made in the state of the cloud.  Congress needs
an interface for cloud services that allows it to execute actions that
change the state of the cloud.


Proposed change
===============

This change will create a new (mix-in) class called ExecutionDriver that has
one main function: execute(name, positional-args, named-args).  That interface
takes the name of the action, a list of its positional arguments, and a
dictionary of its named arguments and executes that action on the cloud
service.

This change will also overload the functionality of

policy/runtime.py:Runtime.execute()

in

policy/dseruntime.py:DseRuntime.execute()

so that a call to 'execute' by the policy engine will send a DSE message to
the appropriate service to execute the ExecutionDriver:execute() function.
This will be a unicast message from the policy engine to the appropriate
service on the DSE.  That service will be included in the name of the action,
e.g. 'nova:disconnectNetwork' is an action that is sent to the 'nova' service.

As a starting point, this change will include an implementation of the
ExecutionDriver mixin within datasources/nova_driver.py
(or another service that the primary assignee chooses).


Alternatives
------------

An alternative is to make the ExecutionDriver class not a mix-in but
a standalone class, thereby having two separate files for each
cloud service, e.g. nova_datasource_driver.py and nova_executor_driver.py.

Treating the ability to execute a service as an interface is beneficial
because in reality services typically have some functionality that allows
them to read the state of the cloud and other functionality that allows
them to change the state of the cloud.  Treating them as different files
makes them seem to be separate services.

Another alternative choice in the design is to use publish/subscribe to
communicate action execution between the policy engine and the instance
of ExecutionDriver.   Publish/subscribe has the drawback that it could
result in multiple services subscribing to the same action (perhaps by
accident) and a single 'execute' command producing several API calls
modifying the state of the cloud.  The unicast choice eliminates this
as a potential problem (though admittedly eliminates the possibility
of a use case where there are legitimately 2 different services that
need to be informed of the action).


Policy
------

None

Policy Actions
--------------

This is a step toward reactive enforcement.


Data Sources
------------

Eventually all data sources will want to implement the ExecutionDriver
interface.


Data model impact
-----------------

None.


REST API impact
---------------

We will want to expose the policy/dse_runtime.py:DseRuntime.execute()
command via the API.

Question: Should we rename the "data-sources" in the API to "cloud-services"?


If so, then we'd have...

Execute an action on a specified service. ::

 POST v1/cloud-services/nova?action=execute -d {'name': 'disconnectNetwork',
                                                'args': ['vm123', 'net456'],
                                                'options': {}}



Execute an action as if the given policy dictated it should be executed. ::

 POST v1/policies/alice?action=execute -d {'name': 'nova:disconnectNetwork',
                                           'args': ['vm123', 'net456'],
                                           'options': {}}


These API calls will have an empty response (HTTP code 204 perhaps) or an
appropriate error response if there was an error raised *before* the API call
was actually made.  These API calls will return before the call we are
executing returns.



Security impact
---------------

This change gives anyone with the ability to write policy the ability to
execute API calls using the same rights Congress has been granted.  Since
we typically run Congress with administrative rights, we need to secure
Congress authentication properly.  But that is already accomplished because
we use the standard Keystone authentication system.


Notifications impact
--------------------

None

Other end user impact
---------------------

python-congressclient would need a new endpoint if we expose the execute()
functionality via the API.


Performance impact
------------------

Because the DatasourceDriver and ExecutionDriver will most often be implemented
as part of the same DSE instance, a poorly written ExecutionDriver could
impair the ability of that driver to properly read information via the
DatasourceDriver interface.  The two pieces of functionality
run in a single thread, i.e. either one runs or the other runs.  This is
actually beneficial because we would not want the driver to be pulling new
data at the same time it executes a change in that data.

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
  <launchpad-id or None>

Other contributors:
  <launchpad-id or None>

Work items
----------

- Rename folder congress/datasources to congress/cloudservices and change
  etc/congress/datasources.conf.sample appropriately.  There may be other
  places that reference 'datasources' explicitly.

- Add congress/cloudservices/execution_driver.py to include the mixin class
  ExecutionDriver

- Implement policy/dseruntime.py:DseRuntime.execute() to send a unicast
  message to the appropriate service or raise an error if that service does
  not exist on the bus.

- Implement the ExecutionDriver for an existing service, e.g. Nova.


Dependencies
============

None


Testing
=======

- Unit tests that ensure a call to DseRuntime.execute() invokes the appropriate
  ExecutionDriver.execute().

- Tempest tests that invoke execute() and ensure the proper change actually
  happens.



Documentation impact
====================

Need to add description of actions and action format to docs, along with
documentation for execute().


References
==========

None
