..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================================
Z3 as an alternative Datalog engine
===================================

https://blueprints.launchpad.net/congress/+spec/alternative-engine-z3

Z3 is an open source Datalog engine developed by Microsoft.
We propose an extension of Congress that lets the user define
theories (ie. set of rules producing new tables from existing ones
that can be later used by other theories) that will be treated
by the Z3 engine instead of the existing one.

Problem description
===================

A cloud operator may want to check if connectivity exists between two machines
or if they are effectively isolated. This means understanding how traffic is
routed (ie. what are the path the traffic may take) and how security groups and
firewall rules are applied along those path.

Here are the main characteristics needed by the Datalog engine for
solving such problems:

* Recursion: routing packets means building paths (link paths, networks)
  and paths are naturally described recursively.

* Efficient representation of IP addresses and IP address ranges as commonly
  found in switching devices and firewalls.

* Efficient handling of large volumes of data.

Proposed change
===============

We propose to integrate Z3, Microsoft open-source automatic prover, as an
alternate Datalog engine for Congress.

Z3 theories will be expressed in the same syntax as standard theories.
But the Datalog language supported by these theories will be tailored to what
Z3 supports in Datalog mode:

* different and reduced set of built-ins as explained in the Policy section,

* no modals.

Architecture
------------

Overview
^^^^^^^^
A new kind of theory will be introduced alongside the classical
NonRecursiveTheory. Z3 theories will consume data from
other theories and use the Z3 engine to apply rules.

There will be one single Z3 solver instance per analyzer that will be
abstracted in Python as a Z3 context object. Z3 context will host all tables
of Z3 theories in Z3 format.

Z3 theories should be callable from other theories in particular from
regular non-recursive theories.

Z3 theories can use data from other theories. The external tables used by
Z3 theories will be completely imported in the Z3 context.

Compilation of Z3 theories will be delayed until queries are done on those
theories. We will regularly update the representation of external tables in
the Z3 engine to keep them synchronized. Z3 does not support undoing relations
or rules, so a new context must be created each time a change occurs.

Type-checking
^^^^^^^^^^^^^

Z3 requires typed data. Type-checking will be completely implicit and we will
not add type-constraints in the language.

The type-checker will preserve the types from external tables. It will raise
an error if a variable should have two different types according to two
different tables.

The type-checker will try to give the most precise type to constants that
appear in Datalog programs to make them compatible with external tables.
For example the value ``Ingress`` is usually typed as a string (``Str`` type)
but in the context of a Neutron security group, it will be given the type
``Direction``

To be able to have two representations for a value, we will introduce
explicit converter built-ins: ``builtin:cast(x, 'Direction', y, 'Str')``
will convert a variable ``x`` of type ``Direction`` in a variable ``y``
of the standard string type. It will be implemented as a table where each
row is a pair of the representations in the two different types of a same
object. Those tables will be filled when constants in programs are compiled
and external tables are imported.

The same trick will be used to have a bit field representation of Ip addresses.

Finally the type-checker should support some kind of generic polymorphism for
built-ins. For example equalities and comparison should work as long as the
compared objects are of the same type, addition of two items belonging to a
given subtype of integers should give back an item in the same subtype. We do
not plan to support user-defined polymorphic tables. For example the following
rule is ill defined: ::

    add3(x,y,z,t) :- builtin:plus(x,y,u), builtin:plus(u,z,t)

It may be desirable to have type-constraints in the language to constrain
such variables.

Internal representations
^^^^^^^^^^^^^^^^^^^^^^^^
Internally Z3 will use bit-vectors. Integers will be translated in 32 bits
bit-vectors. Strings will be bijectively associated with an integer and
represented by its associated bit-vector.

When possible we will use finer types (enumerations) that require smaller
bit-vector.

Alternatives
------------

We could try to rephrase the theories to fit the current engine. For example we
could unroll recursive calls up to a reasonable depth (packets have a TTL).
Unfortunately this solution does not scale well. Unrolling recursive calls
change in fact the overall complexity of the program. Experiments with octant
have shown that some problems that require several minutes of computation with
the current engine can be solved in tenth of seconds with octant.

We could try to incorporate Z3 algorithmic in Congress engine: this would be
time consuming and hazardous. Z3 uses very efficient C++ data structures to
represent tables and more that ten years have been devoted to its development.

We could use another external engine: there may be cases where another Datalog
engine could be a better candidate. The new Congress architecture with explicit
data-type has been designed to let several engines share a same pool of data.
The strong points of Z3 are its overall efficiency and its data-structures
specially crafted for network related problems.

There may be cases where we may wish to have several Z3 contexts with different
settings. Contexts define the variant of the Z3 engine in use and different
theories may benefit from different settings.


Policy
------

Here are the main characteristics of Z3 policies:

* Adds support for recursive rules.

* Support of built-in predicates for equalities / comparison

* Support for basic bit arithmetic (to be used with IP address).

* New built-ins for converting opaque IpAddress to bit fields and IpNetwork
  to a pair of bit-fields (mask and address).

* Built-ins to convert a value that exists in different types from one
  representation to the other.

The last two kinds of built-ins will in fact be implemented by tables computed
when data-sources and programs are synchronized.

Here is an example of policy that verifies if two networks are connected by
a chain of routers and if VMs are attached to two networks that belong to two
sets of networks that should be kept isolated: ::

    connect1(X) :- neutronv2:networks(id=X, name="N1")
    connect2(X) :- neutronv2:networks(id=X, name="N2")

    linked(X,Y) :-
        neutronv2:ports(network_id=X, device_id=Z), neutronv2:routers(id=Z),
        neutronv2:ports(network_id=Y, device_id=Z)

    path(X,Y) :- linked(X,Y)
    path(X,Y) :- path(X,Z), linked(Z,Y)

    interco_error(X,Y) :- connect1(X), connect2(Y), path(X,Y)

    network1(X) :- connect1(Y), path(Y, X)
    network2(X) :- connect2(Y), path(Y, X)

    double_attach(X) :-
        nova:server(id=X),
        neutronv2:ports(network_id=X, device_id=Y), network1(Y),
        neutronv2:ports(network_id=X, device_id=Z), network2(Z)

Policy actions
--------------

None. Z3 theories will not support actions. Support may be added later.

Data sources
------------

Independent from the data-source used.


Data model impact
-----------------

None


REST API impact
---------------

We may want to move to more atomic definitions of theories. A theory is
a complete program and should be considered as such. Adding and removing
rules on the fly makes compilation and type-checking unnecessarily complex.

Security impact
---------------

None

Notifications impact
--------------------

None

Other end user impact
---------------------

For python-congressclient, when a new theory is created, there is a new
kind alternative: z3.

Performance impact
------------------

Well behaved Z3 theories will be much faster than equivalent non-recursive
theories. But programs that require too much computing power may be harder to
abort on the Z3 engine.

Other deployer impact
---------------------

Z3 is not a pypi package. There is an unofficial z3solver package but it has
some limitations (not an up-to-date version and a version emitting a lot of
spurious messages). For some Linux distributions, there is no pre-compiled
version of Z3 available. Z3 must be compiled from source on those systems.

Due to all these constraints, we have chosen NOT to add z3 as a requirement.
z3theory will be available IF z3 has been deployed on the server hosting
congress.


Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------


Primary assignee:
  pcregut

Other contributors:


Work items
----------

For Rocky we do not plan to add built-ins. The basic Datalog language is
sufficient for interesting examples.

During Stein iteration, built-ins and the treatment of polymorphism
in the type-checker will be added.

Dependencies
============

This feature depends on Microsoft theorem prover Z3.

Testing
=======

Unit tests will be done with a mock of z3 library.

For integration tests, new settings will be added to devstack so that Z3
can be either installed from source (rather slow) or directly deployed from
a pre-compiled release (only available for Ubuntu 14.04, 16.04 and Debian
8.10). As the python package is deployed globally, tempest may skip Z3 tests
if it cannot import the python package.

Documentation impact
====================

Documentation should at least mention:

* How to enable Z3,

* Z3 theories and the kind of builtins supported,

* Explain the benefit of recusive theories.

References
==========

[1] https://github.com/Z3Prover/z3

[2] https://github.com/Orange-OpenSource/octant

[3] https://specs.openstack.org/openstack/congress-specs/specs/rocky/explicit-data-types
