..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Policy creation in Horizon
==========================================

https://blueprints.launchpad.net/congress/+spec/horizon-create-policies

Congress policies can be created from the command line interface (CLI). Provide
a way to create policies from the Horizon dashboard that helps users who have
little or no experience with writing Datalog.


Problem description
===================

Users unfamiliar with Datalog may find it challenging to write rules for a
Congress policy.


Proposed change
===============

Add a way to create Congress policies in the Policies panel in Horizon that
does not require users to have knowledge of Datalog to write policy rules.
Something like the draggable blocks in Node-RED or BipIO could be moved around
and connected together by the user to construct a policy rule. Or have a
template that presents the Datalog pieces of a rule in a natural language
format.


Alternatives
------------

One alternative is to enhance the CLI to assist users in writing policy rules.
However, a graphical interface can have interactive mechanisms to help users
navigate the structure of a policy rule that a CLI cannot provide. For example,
rule components could be more easily represented with graphics in a web
interface vs. a CLI-based wizard, which is limited to text representations.


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

Caution must be exercised when using data entered by a user. An example of a
type of security issue is SQL injections, if the user data is going to be
inserted into a database. User data should be sanitized to mitigate such risks.


Notifications impact
--------------------

None.


Other end user impact
---------------------

The end user will be able to go to the Policies panel in Horizon and use some
knobs to write policy rules and create a policy in Congress.


Performance Impact
------------------

None.


Other Deployer Impacts
----------------------

None.


Developer Impact
----------------

None.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  jwy

Other contributors:
  <launchpad-id or None>

Work Items
----------

- Knobs and form to enter policy details and rules.
- Logic (e.g., autocompletion) to fill in policy table names, data source table
  names, etc., that can be used in a policy rule.
- Backend to pass information on to Congress for policy creation.
- Form to enter Datalog directly for a rule, for users who are more comfortable
  with the language.

Keep in mind that we may want to use the same UI for updating policies later.


Dependencies
============

None.


Testing
=======

Additional Tempest tests are not needed because the Congress code is not being
modified here.


Documentation Impact
====================

None.


References
==========

Datalog Policy Language:
https://github.com/stackforge/congress/blob/master/doc/source/policy.rst#2-datalog-policy-language

Node-RED:
http://nodered.org

BipIO:
https://bip.io
