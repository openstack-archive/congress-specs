..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========================
Support version list API
========================

https://blueprints.launchpad.net/congress/+spec/api-version

The Congress API should be able to list current supported API versions.


Problem description
===================

Users want to talk with Congress API, but no API support to query the supported
versions, so users have no idea about which versions can be supported in the
current Congress deployment.


Proposed change
===============

Export the version list API, and show the current API details, include: id,
status, update time, links, like other OpenStack project: nova, neutron and
etc. Although Congress only support one version API(v1) currently, but the
version list API make sense for future API evolution.


Alternatives
------------

None


Policy
------

None


Policy actions
--------------

None


Data sources
------------

None


Data model impact
-----------------

None


REST API impact
---------------

* Specification for the method

  * List the current supported API versions.

  * Method type: GET

  * Normal http response code(s): 200

  * Expected error http response code(s): None

  * ``/``

  * Parameters which can be passed via the url: None

  * JSON schema definition for the body data if allowed: None

  * JSON schema definition for the response data if any:

    ::

    {
      "type": "object",
      "properties": {
          "versions": {
              "type": "array",
              "items": {
                  "type": "object",
                  "properties": {
                      "status": {
                          "type": "string"
                      },
                      "updated": {
                          "type": "string"
                      },
                      "id": {
                          "type": "string"
                      },
                      "links": {
                          "type": "array",
                          "items": {
                              "type": "object",
                              "properties": {
                                  "href": {
                                      "type": "string"
                                  },
                                  "rel": {
                                      "type": "string"
                                  }
                              },
                              "additionalProperties": false,
                              "required": ["href", "rel"]
                          }
                      }
                  },
                  "additionalProperties": false,
                  "required": ["status", "updated", "id", "links"]
              }
          }
      },
      "additionalProperties": false,
      "required": ["versions"]
    }

* Example use case:

  ::

    GET /
    {
      "versions": [{
          "status": "CURRENT",
          "updated": "2015-07-03T11:33:21Z",
          "id": "v1",
          "links": [{
              "href": "http://10.250.10.29:1789/v1/",
              "rel": "self"
          }]
      }]
    }

* There should not be any impacts to policy.json files for this change.


Security impact
---------------

None

Notifications impact
--------------------

None

Other end user impact
---------------------

The related works in python-congressclient will also be added.

After this modification, user could get the API version details, like this:

  ::

  openstack congress version list


Performance impact
------------------

None

Other deployer impact
---------------------

We modify api-paste.ini to add some stuff, so if the operator prepare to
upgrade from old release, he need to add the new config items to old
api-paste.ini file or override the old using new one.

Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:

  Rui Chen <chenrui.momo@gmail.com>


Work items
----------

* Adds Version class to assemble API versions response.
* modify api-paste.ini to route the request to the new logic.
* Make python-congressclient supporting this API.


Dependencies
============

None


Testing
=======

Some unit tests should been added to cover the new API.


Documentation impact
====================

The related content should be added in Congress API document.


References
==========

None
