..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Add Action Listing to api
==========================================

Just as the API allows us to query the available tables that a
datasource provides, it should be possible to query the available
actions that a datasource can execute.

Problem description
===================

Currently, datasources have the ability to execute actions, but there is no api
call to ask what the available actions are.

Proposed change
===============

Add an API call that returns the list of available actions.

This requires adding a method to the
congress/datasources/datasource_driver.py:ExecutionDriver
class that returns the list of actions that the datasource will execute.

It would be nice if the ExecutionDriver also enforced that it only
executed the actions in that list.

One might consider trying to auto-generate the list of actions based
on the methods available in the python-client being used (for those
datasources using a python-client).

Alternatives
------------

The alternative is to provide no API and ask people to consult the
documentation to find the list of available actions.

This is more
attractive than it sounds because it gives us the freedom to
execute any action that the python-client for the underlying
datasource exposes.  If someone adds a new method for the python-client
then a policy writer immediately gets access to that method, without
requiring any changes in Congress.

The downside to this approach is that end-users need to understand
which python client they are using within each datasource driver and
then look up the docs for that python client.  So in terms of usability,
it's pretty terrible.


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

There should be no data-model impact, since the added information is
implemented as part of the datasource driver, not stored in the database.

REST API impact
---------------

GET /v1/data-sources/<datasource-id>/actions

Should error if the datasource does not implement the proper method.

Result should be an array of dicts, where each dict describes an action:
  [{'name': <action-name>,
    'args': <list-of-parameter-names>}]

The assignee is free to extend the fields of an action to include additional
meta-information, such as a description of what the action does, or a
description of what each of the parameters are.


Security impact
---------------

N/A

Notifications impact
--------------------

N/A

Other end user impact
---------------------

Need to add similar functionality to the python-congressclient.

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
  <launchpad-id or None>

Other contributors:
  <launchpad-id or None>

Work items
----------

0. Modify API design doc (linked from the wiki)
1. Add method to ExecutionDriver in congress/datasources/datasource_driver.py
2. Add call handler in congress/harness.py
3. Add route call in congress/api/router.py
4. Add API implementation to congress/api/action_model.py
5. Update python-congressclient

Dependencies
============

N/A

Testing
=======

- Unit tests for API to congress/tests/test_congress.py
- Unit tests for ExecutionDriver


Documentation impact
====================

Add API call

References
==========

N/A
