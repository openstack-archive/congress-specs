..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Add in list support for hdict
==========================================

https://blueprints.launchpad.net/congress/+spec/add-in-list-support-hdict

This blueprint is to add support for a new translator value called in-list
which is useful in translating a hdict that contains another hdict which
is a list.

Problem description
===================

Several datasources return data in the format of a dict(list(dict()) (for
example neutron ports for fixed_ips)::

    {
        "ports": [
            {
                "status": "DOWN",
                "id": "11ce1474-e395-4bda-b48c-820f0d542acd",
                "fixed_ips": [
                    {
                        "subnet_id": "e56370ba-d255-486c-9907-ad4c6aed0241",
                        "ip_address": "10.2.0.2"
                    }
                ]
            }
        ]
    }

congress currently does not have an easy way to deal with data in this format
unless an extra table is created. This blueprint is to provide an
easy generic way to do so without creating the extra table.


NOTE: I only plan to implement this at first for hdict because we believe we
      might be able to remove the vdict type as it might not be needed. If it
      turns out that we need vdict I'll implement this for vdict as well.

Proposed change
===============

To implement this solution I propose we add a new in-list attribute to the
child translator. If present, the datasource driver class will just simply
interate this list and treat each list element as an hdict.

Alternatives
------------

We could force a datasource driver to manually do this themselves though it
would be better to automatically handle this for a user.

Policy
------

N/A

Policy Actions
--------------

N/A


Data Sources
------------

N/A


Data model impact
-----------------

New element 'in-list' can be specified in the translator.


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

Implement this feature.

Dependencies
============

* This needs to be done before we can refactor the neutron driver to remove
  the subtables that it creates.


Testing
=======

Unit tests.

Documentation impact
====================

Will document.

References
==========

N/A
