..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Example Spec - The title of your blueprint
==========================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/congress/+spec/datalog-column-names

Writing policy can be challenging when tables have many columns.
This spec aims to give end users the ability to write policy by
referencing column names directly in Datalog rules.



Problem description
===================

The main problem is that wide tables (those with many columns) are hard to use.
(a) it is hard to remember what all the columns are,
(b) it is easy to mistakenly use the same variable in two different tables in
the body of the rule,
i.e. to create an accidental join,
(c) changes to the datasource drivers can require tedious/error-prone
modifications to policy.



Proposed change
===============

Today people write policy statements as follows:

error(net) :-
   neutron:networks(x0,x1,net,x3,x4,x5,x6,x7,x8,x9,x10),
   ...

This change enables an alternative syntax where columns are referenced by name
instead
of position.

error(net) :-
   neutron:networks(id=net),
   ...

To implement this change, we will expand the grammar to accept column-name
references and modify the AST constructor to translate the rules from
column-name references back into position references.


Alternatives
------------

There were 2 alternatives described in a ML discussion.

2) Be disciplined about writing narrow tables and write
tutorials/recommendations demonstrating how.

Instead of a table like...
neutron:ports(port_id, addr_pairs, security_groups, extra_dhcp_opts,
binding_cap, status, name, admin_state_up, network_id, tenant_id, binding_vif,
device_owner, mac_address, fixed_ips, router_id, binding_host)

we would have many tables...
neutron:ports(port_id)
neutron:ports.addr_pairs(port_id, addr_pairs)
neutron:ports.security_groups(port_id, security_groups)
neutron:ports.extra_dhcp_opts(port_id, extra_dhcp_opts)
neutron:ports.name(port_id, name)
...

People writing policy would write rules such as ...

p(x) :- neutron:ports.name(port, name), ...

[Here, the period e.g. in ports.name is not an operator--just a convenient
way to spell the tablename.]

To do this, Congress would need to know which columns in a table are sufficient
to uniquely identify a row, which in most cases is just the ID.

Pros:
(i) this requires only changes in the datasource drivers; everything else
remains the same
(ii) still leveraging database technology under the hood
(iii) policy is robust to changes in fields of original data

Cons:
(i) datasource driver can force policy writer to use wide tables
(ii) this data model is much different than the original data models
(iii) we need primary-key information about tables

3) Enhance the Congress policy language to handle objects natively.

Instead of writing a rule like the following ...

p(port_id, name, group) :-
neutron:ports(port_id, addr_pairs, security_groups, extra_dhcp_opts,
binding_cap, status, name, admin_state_up, network_id, tenant_id,
binding_vif, device_owner, mac_address, fixed_ips, router_id, binding_host),
neutron:ports.security_groups(security_group, group)

we would write a rule such as
p(port_id, name) :-
neutron:ports(port),
port.name(name),
port.id(port_id),
port.security_groups(group)

The big difference here is that the period (.) is an operator in the language,
just as in C++/Java.

Pros:
(i) The data model we use in Congress is almost exactly the same as the data
model we use in Neutron/Nova.

(ii) Policy is robust to changes in the Neutron/Nova data model as long as
those changes only ADD fields.

(iii) Programmers may be slightly more comfortable with this language.

Cons:
(i) The obvious implementation (changing the engine to implement the (.)
operator directly is quite a change from traditional database technology.
At this point, that seems risky.

(ii) It is unclear how to implement this via a preprocessor (thereby
leveraging database technology).  The key problem I see is that we would need
to translate port.name(...) into something like option (2) above.  The
difficulty is that TABLE could sometimes be a port, sometimes be a network,
sometimes be a subnet, etc.

(iii) Requires some extra syntactic restrictions to ensure we don't lose
decidability.

(iv) Because the Congress and Nova/Neutron models are the same, changes to
the Nova/Neutron model can require rewriting policy.




Policy
------

error(net) :-
   neutron:networks(id=net)


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

String-arguments to API calls representing policy can use either
position or column syntax.

Security impact
---------------

None

Notifications impact
--------------------

None

Other end user impact
---------------------

Other UIs can expose this enhanced syntax, but since currently all UIs
just pass policy statements as strings, they will require no actual changes
to leverage the enhancement.

Performance impact
------------------

Should have no performance impact, with the possible exception that
eventually we will want to reverse the preprocessing step for tracing
so that we present users with a more intuitive trace.

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
  thinrich


Work items
----------

* Modify grammar
* Make datasource schemas available to runtime
* Add preprocessor to rule AST constructor to convert column references into
  positional references.


Dependencies
============

The following change makes datasource schema available to policy engine

Change-Id: I7cfbd82c721509634c0acfd51e66031af2ed7f2d


Testing
=======

No tempest tests are necessary.  Unit tests only.


Documentation impact
====================

Should include modifications to docs to simplify the examples that use
tables with many columns.  Will also require modifying the section where
we introduce Datalog.


References
==========

Mailing list discussion:
* http://lists.openstack.org/pipermail/openstack-dev/2014-August/041862.html

