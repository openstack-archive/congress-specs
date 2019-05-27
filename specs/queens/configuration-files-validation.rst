..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========================================
OpenStack Configuration Files Validation
========================================

https://blueprints.launchpad.net/congress/+spec/configuration-files-validation

Congress could be used by cloud operators to formalize the constraints between
options defined in configuration files with the help of business rules written
in Datalog and verify the compliance of their deployments automatically.
It is intended to be complementary to config management systems
that should only create valid configuration files
and to integration tests such as RefStack.


Problem description
===================

Each OpenStack service on each node in parameterized by a set of configuration
files describing the value of the various configuration options. In this
proposal we only consider configuration files managed by the oslo-config
library. Configuration options are not independent:

* Each option value is constrained but the constraints may go beyond the
  expressive power of oslo-config type system.

* For a given service on a given node, constraints may exist between different
  options.

* On a given node, constraints and incompatibilities may exist between various
  options belonging to different services. For example, when a functionality in
  a given service depends on the availability of another functionality in
  another service.

* Between nodes, other constraints exist either for a given service or between
  services.

Constraints may be defined by different actors:

* Service developers know the constraints on the options they define and
  usually document them informally in the source code in the description field
  associated to the option.

* Cloud integrators will discover additional constraints when they develop
  deployment code. Most of the time those options are only implicitly defined
  in the source code of the scripts.

* Administrators may want to enforce additional constraints reflecting the
  operational environment constraints.

* For companies having multiple instances of OpenStack, rules could be
  used to ensure compliance of the instances with their global engineering
  rules.

* Although OpenStack provides abstractions that hide implementation details
  from end-users, some back-ends or some back-end combinations may impose
  constraints on what is available to the end-user. When administrators
  discover some back-end limitations, it may be useful to add congress rules
  reflecting those limitations before a more permanent solution is found.

Datalog could be used as a formal language to describe those constraints and
Congress could be used to enforce those constraints if the configuration files
were available for processing. Although Congress is usually used for more
dynamic data coming from running instances, nothing precludes the use of more
static data sources.

Proposed change
===============

The extension adds a new agent deployed on OpenStack nodes that collect the
configuration files as defined in its own configuration. The agent communicates
with a datasource driver included in the congress analyzer through the
message bus. The agent is configured with its own configuration file describing
the files it transmit (an example configuration file is given in the 'Other
deployer impact' section).

The agent is responsible for conveying information on option values but also on
their meta-data: layout in groups, type, flags such as deprecation, secret
status, etc. Meta-data are described in templates which are in fact
a collection of namespaces. Namespace contain the actual definition of
meta-data. To avoid versionning problems, Meta-data must be obtained directly
from the service.

To limit the amount of traffic between the driver and agents, template files
are hashed. Agents first reply to queries with hashes. The driver will request
the content only if the hash is unknown.
The same process is used for namespaces.

The only processing performed by agents on the option files is the removal of
the values of secret options.

The datasource driver is responsible for populating the extensional database
with information provided by the agents:

* option definitions are extracted from templates and namespaces and translated
  in new Datalog tables.

* option values are parsed by the oslo-config library and stored with an
  identification of the origin (file and node).

The Datalog model defines several tables:

* hosts

* files

* templates

* namespaces

* option values

The definition of an option is split between three tables:

* The option table defines the mapping between the option id and the fields
  that build up the key: name, namespace and group of the option.

* The option_info table defines the type and the flags of the option. It uses
  the id defined in the option table to uniquely identify the option.

* Depending on the type of the option, attributes specific to a given type are
  defined in a specialized table. The option id is used again as a key to refer
  to the option table.

Regarding the uniqueness of configuration meta-data in the extensional
database, the driver must ensure that the ids are deterministic. An option
identified by the same name, same group name and same namespace name should
always be given the same unique id. The use of the MD5 hash function (there is
no cryptographic requirement) guarantees uniqueness and determinism.

Alternatives
------------

Installers usually generate correct set of configuration files for a
more or less extensible set of options but provide little help for live
maintenance of those files. They may also restrict the configuration
space beyond what is necessary.

The proposed change must be considered as a work in progress. It does not fully
address the problem of the location and the management of constraints. Most
constraints are known by the service developers and should be maintained in the
source code of services in a dedicated meta-data field of
the option definition.

The use of an external agent to push a service configuration is not the only
solution. The oslo-config library could be modified to push the configuration
read by the service to the datasource driver. This could be done through
the use of a hook in oslo-config. It would require additional configuration of
the services to identify the endpoint.

Policy
------

We would like to aggregate policy violations in a few tables partitioning
violations by their degree (error, warn, info).

Example:

X: neutron-server host

Y: ovs-agt host

Z: lb-agt host

::

    warn('Multinode OVS and linuxbridge use incompatible UDP ports',
    'vxlan_conflicting_ovs_lb_udp_ports') :-
        vxlan_conflicting_ovs_lb_udp_ports()

    vxlan_conflicting_ovs_lb_udp_ports(Y, Z) :-
        value(X, 'neutron.conf', 'DEFAULT', 'core_plugin', 'ml2'),
        value(X, 'ml2_conf.ini', 'ml2', 'type_drivers', 'vxlan'),
        value(X, 'ml2_conf.ini', 'ml2', 'mechanism_drivers', 'openvswitch'),
        value(X, 'ml2_conf.ini', 'ml2', 'mechanism_drivers', 'linuxbridge'),
        value(Y, 'openvswitch_agent.ini', 'agent', 'tunnel_types', 'vxlan'),
        not value(Y, 'openvswitch_agent.ini', 'agent', 'vxlan_udp_port', 8472),
        value(Z, 'linuxbridge_agent.ini', 'vxlan', 'enable_vxlan', 'True')


We'd like the 'value' table to be intelligible to most potential contributor,
and also meaningful enough to spare them from predictable joins.

Currently the 'value' table is derived from the extensional tables, which
describe configuration files and their meta-data. Although it is defined
intentionally, it could be useful to consider it as an extensional predicate.
Here is how we derived it:

::

    value(hostname, file, group, name, value) :-
        config:option(id=option_id, group=group, name=name),
        config:file(id=file_id, host_id=host_id, name=file),
        config:host(id=host_id, name=hostname),
        config:value(option_id=option_id, file_id=file_id, val=value)

Policy actions
--------------

None.

Data sources
------------

The data sources are the different configuration files of OpenStack projects
using the Oslo.config library for their configuration.

Data model impact
-----------------

None

REST API impact
---------------

None

Security impact
---------------

Configuration files contain sensitive credentials. Those credentials MUST NOT
be transmitted to the Congress engine. The agent has access to types and must
filter out credentials. Values of any option marked as secret  will not be
available within the engine.

Notifications impact
--------------------

None

Other end user impact
---------------------

None other than the usual management of the datasource and policy.
Eventually, we would like to feed the engine with rules that are coming from
and maintained in the services source code.

Performance impact
------------------

Performance impact should be limited because of the static nature of the values
provided by this datasource.

The main impact is the traffic on the message bus used to exchange
configuration files between agents and Congress server. The server may
rate-limit this traffic. If performance is still a concern, another solution is
to limit the use of the bus to announcement and use a REST endpoint on the
server to record new configuration files.

We give the driver control over the way data is retrieved. We want to prevent
the duplicated sending of files and templates, and to prevent overloading the
driver. When the driver is activated, it periodically notifies agents, over the
communication bus requesting their data description. An agent send description
of the files it has been set to provide. The description contains hashes of
namespaces, templates and configs. The driver then requests the resources,
which hashes have not been recognized.

We use the RPC server of the datasource associated DseNode.

Other deployer impact
---------------------

We add a dedicated group and options to configure an agent and the
configuration files to manage.

*validator.host*

A string option serving as a node id, in a way that is meaningful to
administrators.

*validator.version*

A string option introducing the notion of version and what it would be on this
node. It could be used to discriminate handling of different version of a
config-file coexisting in a cloud instance during a migration.
This information may be provided differently in the future to be defined
at the services level.

*validator.services*

An dict option describing the OpenStack services activated on this node. The
values are also dictionaries. Keys are paths to config-files,
values are paths to the associated templates. For instance::

    congress: { /etc/congress/congress.conf:
    /opt/stack/congress/etc/congress-config-generator.conf }


Example config :

::

    [DEFAULT]
    transport_url = rabbit://..@control:5672/

    [validator]
    services = nova : { /etc/nova.conf:
    /opt/stack/nova/etc/nova/nova-config-generator.conf:
    version = ocata
    host = node_A

Developer impact
----------------

Discuss things that will affect other developers working on OpenStack,
such as:

* If the blueprint proposes a change to the driver API, discussion of how
  other hypervisors would implement the feature is required.


Implementation
==============

Assignee(s)
-----------

Primary assignees:

* Valentin Matton

* Pierre Crégut


Work items
----------

* Agent to collect the config files

* Datasource interacting with the agents

* Integration to devstack


Dependencies
============

We will depend on oslo-config-generator config-files, which can be used to
describe OS services config-files.
https://github.com/openstack/oslo-specs/blob/master/specs/juno/oslo-config-generator.rst

We rely extensively on the oslo-config lib.

Testing
=======

We propose to use tempest tests in a setting with 2 nodes. One of which hosts
the congress-server and the second an agent. Communications will be tested :
sending of meta-data and files.

Documentation impact
====================

This feature introduces an agent component that requires separate
configuration. It also defines new datasources.


References
==========

