..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================
Explicit Data Types
===================

https://blueprints.launchpad.net/congress/+spec/explicit-data-types

In preparation to support more policy engines in addition to the standard
Congress policy engine (e.g., ones based on SQL, influxDB, or Z3), we need
explicit typing of table columns in the data sources and the
data service engine. This spec proposes to support explicit typing, including
custom types, in an extensible, evolutionary, and backwards-compatible manner.


Problem description
===================

Because every rule engine has limits in expressiveness and performance, there
is no one-size-fits-all rule engine that works well for all use cases. For
this reason, Congress was architected from the beginning to support multiple
policy engines in addition to the standard ``agnostic`` engine.

One problem is that Congress deals with untyped table columns, which
conflicts with many candidate bases for policy engines (e.g., SQL, influxDB,
Z3).

To have an open framework in which multiple policy engines and mulitple
data sources work well together, we need an explicit type system that is
extensible (to fit the particular needs for different data sources and
different policy engines), loosely-coupled (between different data sources and
policy engines), and backwards-compatible with the standard ``agnostic`` engine
and existing data source drivers.

As an illustration of the need for extensibility and loose-coupling, consider
that a SMT-based Z3 prover needs types that are as narrow as possible
(e.g., enum{'ingress', 'egress'}), but another engine does not understand
such narrow types. A data source driver then must specify its types
in a way that works well for both engines.


Proposed change
===============

Overview
--------
The basic idea is that we let each data source driver describe its data as
precisely and narrowly as practical using as combination of base types
and custom types.
Then each policy engine is responsible for using that information to represent
the data using its internal representation. A policy engine would likely not
understand every custom type used by a data source driver (especially because
we expect new ones to be introduced over time). To allow a policy engine to
handle custom data types it does not "understand", we require that every
custom data type inherit from another data type, all of which ultimately
inherit from one of the base types all policy engines must handle. That way,
when a policy engine encounters a value of a custom type
it does not understand, the policy engine can fall back to handling the value
according to an ancestor type the engine does understand.

What is a type?
---------------

To define what a type is in this spec, we first establish four related
concepts:

#. *source representation*: the representation used in the data received from
   an external source. Each external data source has its own source
   representation defined outside of Congress. In an IP address example, one
   source could use IPv4 dotted decimal string ``"1.0.0.1"`` while another
   source could use IPv6 (short) hexadecimal string ``"::ffff:100:1"``.

#. *internal representation*: the representation used by a Congress service
   (e.g., a policy engine). Each Congress service can use a different internal
   representation. For example, the IP address may be represented as a string
   in one engine, as an integer another engine, and as a bit-vector in yet
   another engine.

#. *exchange representation*: the representation used as the data is published
   and received among Congress services. In the IP address example, this could
   be the integer ``0x01000001``.

Data is received from an external data source in the form of the
*source representation*. The data source driver converts from the source
representation to the *exchange representation* before publishing the data to
Congress services such as the policy engine. A Congress service such as a
policy engine will process the data and use some form of
*internal representation* for storing the data for future use.

The primary purpose of a Congress data type is to be an exchange format
specification used between the various Congress data sources and services each
with their own representations. The developers of each Congress service
can use the Congress data type of a piece of data or a table column to
know what exchange format to expect and what is being represented. For this
purpose, a Congress data type can be thought of as a tag associated with each
piece of data or table column.

As a secondary convenience, each Congress data type also provides helpful
methods for processing a value of that type. At a minimun, the Congress
data type would provide a method for validating a value as valid for that
type.

Finally, the entire collection of Congress data types cannot be anticipated
upfront. So it is impractical for each Congress data consumer to be programmed
to recognize and handle every Congress data type. We need a more structured
and extensible collection of "tags". We organize the Congress data types
in an is-a hierarchy. First there are a number of base types which all
Congress data consumers must recognize and handle. All additional non-base
types inherit from a base type or another non-base type in a well-founded
hierarchy. Then, each data consumer can handle all data types simply by
treating each piece of data according to its base type. Optionally, a data
consumer can add special handling of certain non-base types.

As an example of inheritance, the IP address type could inherit from the
general string type so that policy engines which do not specifically
handle the IP address type will handle the data as a general string.

Putting all three of the above pieces together, we define a Congress data type
as a subclass of the following ``CongressDataType`` abstract base class.

.. code-block:: python

  @six.add_metaclass(abc.ABCMeta)
  class CongressDataType(object):

      @classmethod
      @abc.abstractmethod
      def validate(cls, value):
          '''Validate a value as valid for this type.

          :Raises ValueError: if the value is not valid for this type
          '''
          raise NotImplementedError

      @classmethod
      def _get_parent(cls):
          congress_parents = [parent for parent in cls.__bases__
                              if issubclass(parent, CongressDataType)]
          if len(congress_parents) == 1:
              return congress_parents[0]
          elif len(congress_parents) == 0:
              raise cls.CongressDataTypeNoParent(
                  'No parent type found for {0}'.format(cls))
          else:
              raise cls.CongressDataTypeHierarchyError(
                  'More than one parent type found for {0}: {1}'
                      .format(cls, congress_parents))

.. important::
  These classes are not expected to be instantiated as wrapper objects, but
  they merely act as organizing units of these class methods that serve to
  describe and manipulate the values as well as encode the data type hierarchy.
  The reason why classes are used is two fold.

  1. The Python inheritance hierarchy is a convenient way to declare and access
     the relationship between the Congress data types. Each non-root concrete
     subclass (A) of CongressDataType inherits from exactly one other concrete
     subclass (B) of CongressDataType to encode that (A) is a subtype of (B)
     in the data type hierarchy. A concrete subclass of CongressDataType may
     inherit from multiple parent Python classes, but among the parents there
     must not be more than one subclass of CongressDataType.

  2. Python inheritance makes it convenient for a Congress data type to reuse
     the code in a parent Congress data type.

Further details
---------------

Specification of data types
```````````````````````````
Data source drivers may optionally specify explicit data types in its
value translator definition. The data service base class attaches data type
information (table schema) in each table publish.
For flexibility and backwards-compatibility, an explicit type is not required
for every column.

The standard ``agnostic`` engine handles untyped table columns.
Other policy engines may or may not handle untyped tables columns.
When a policy engine that requires typing receives untyped data, the policy
engine may either reject the data or impose typing.

Below is a sample translator definition showing how the explicit data types
could be specified for the ``flavors`` table in the Nova data source driver.

.. code-block:: python

  flavors_translator = {
      'translation-type': 'HDICT',
      'table-name': FLAVORS,
      'selector-type': 'DOT_SELECTOR',
      'field-translators':
          ({'fieldname': 'id', 'desc': 'ID of the flavor',
            'translator': {'translation-type': 'VALUE',
            'data-type': CongressStr}},
           {'fieldname': 'name', 'desc': 'Name of the flavor',
            'translator': {'translation-type': 'VALUE',
            'data-type': CongressStr}},
           {'fieldname': 'vcpus', 'desc': 'Number of vcpus',
            'translator': {'translation-type': 'VALUE',
            'data-type': CongressInt}},
           {'fieldname': 'ram', 'desc': 'Memory size in MB',
            'translator': {'translation-type': 'VALUE',
            'data-type': CongressInt}},
           {'fieldname': 'disk', 'desc': 'Disk size in GB',
            'translator': {'translation-type': 'VALUE',
            'data-type': CongressInt}},
           {'fieldname': 'ephemeral', 'desc': 'Ephemeral space size in GB',
            'translator': {'translation-type': 'VALUE',
            'data-type': CongressInt}},
           {'fieldname': 'rxtx_factor', 'desc': 'RX/TX factor',
            'translator': {'translation-type': 'VALUE',
            'data-type': CongressFloat}})
            }



Union of types
``````````````
When a policy engine derives a new table out from existing tables, it create
a table column which contains data from more than one type. In these cases,
the new column shall be typed as the least common ancestor type of all the
constituent data types. To ensure that a least common ancestor type always
exists, we establish the Congress string type as the root parent of all the
other types.

Types hierarchy
```````````````
This spec proposes a flexible and extensible type framework. The precise
collection of types would be decided in implementation patches and also
evolved over time. As point of reference, we provide below a sample hierarchy
of types below. (Items enclosed in are abstract categories of types.)

- string

  - [bounded length sting]

    - string of length up to N
    - [string enumerations]

      - boolean
      - network direction (ingress and egress)

  - [fixed length string]

    - string of length N

      - UUID

  - decimal (fixed precision rational number represented as a string)

    - integer

      - [integer enumerations]

        - integer between A and B
        - short integer

    - floating point

  - IP address

Data types sample code
``````````````````````
Sample code for a few selected data types are included here for reference.

.. code-block:: python

  class CongressStr(CongressDataType):
      '''Base type representing a string.

      Sample implementation only. Final implementation needs to deal with
      nullability, unicode, and other issues.'''

      @classmethod
      def validate(cls, value):
          if not isinstance(value, six.string_types):
              raise ValueError

  class CongressBool(CongressStr):
      @classmethod
      def validate(cls, value):
          if not isinstance(value, bool):
              raise ValueError

  class CongressIPAddress(CongressStr):
      @classmethod
      def validate(cls, value):
          try:
              ipaddress.IPv4Address(value)
          except ipaddress.AddressValueError:
              try:
                  ipv6 = ipaddress.IPv6Address(value)
                  if ipv6.ipv4_mapped:
                      # as a design decision in standardized format
                      # addresses in ipv4 range should be in ipv4 syntax
                      raise ValueError
              except ipaddress.AddressValueError:
                  raise ValueError

Abstract categories of data types
`````````````````````````````````
Sometimes, a collection of types all follow a certain pattern. In these
cases, we can provide a framework for creating and handling these types in
a more generic way.
For example, many types are characterized by a fixed, finite domain of strings.
In such cases, we can provide a type factory for creating types in this space.
The following is a sample implementation for a type factory function for
a fixed, finite domain of strings.

In addition, we can also provide a mix-in abstract base class that data
consumers can use to handle the data in a more generic way. In this case,
the abstract base class ``CongressTypeFiniteDomain`` stipulates that
each type inheriting from this class must have a class variable DOMAIN
which is a frozenset of the set of values allowed in the type.
A data consumer may use this information to handle the value in a generic
way without recognizing the specific concrete type used.


Sample code included below for reference.

.. code-block:: python

  @six.add_metaclass(abc.ABCMeta)
  class CongressTypeFiniteDomain(object):
      '''Abstract base class for a Congress type of bounded domain.

      Each type inheriting from this class must have a class variable DOMAIN
      which is a frozenset of the set of values allowed in the type.
      '''
      pass


  def create_congress_str_enum_type(class_name, enum_items):
      '''Return a sub-type of CongressStr for representing a string from
      a fixed, finite domain.'''

      for item in enum_items:
          if not isinstance(item, six.string_types):
              raise ValueError

      class NewType(CongressStr, CongressTypeFiniteDomain):
          DOMAIN = frozenset(enum_items)

          @classmethod
          def validate(cls, value):
              if not value in cls.DOMAIN:
                  raise ValueError

      NewType.__name__ = class_name
      return NewType

As an example, a particular data source driver dealing with firewall rules may
use the factory function to create a custom type as follows.

.. code-block:: python

  NetworkDirection = create_congress_str_enum_type(
      'NetworkDirection', ('ingress', 'egress'))

Converting to ancestor types
````````````````````````````
Generally, each descendent type value is directly interpretable as a value
of an ancestor type. Ideally, every policy engine would recognize and support
The only exception is that values of the non-string types
inheriting from CongressStr need to be converted to string to be interprable
as a value of CongressStr type.
The CongressDataType abstract base class can include additional helper methods
to make the interpretation easy. Below is an expanded
CongressDataType definition including the additional helper methods.

.. code-block:: python

  @six.add_metaclass(abc.ABCMeta)
  class CongressDataType(object):

      @classmethod
      @abc.abstractmethod
      def validate(cls, value):
          '''Validate a value as valid for this type.

          :Raises ValueError: if the value is not valid for this type
          '''
          raise NotImplementedError

      @classmethod
      def least_ancestor(cls, target_types):
          '''Find this type's least ancestor among target_types

          This method helps a data consumer find the least common ancestor of
          this type among the types the data consumer supports.

          :param supported_types: iterable collection of types
          :returns: the subclass of CongressDataType
                    which is the least ancestor
          '''
          target_types = frozenset(target_types)
          current_class = cls
          try:
              while current_class not in target_types:
                  current_class = current_class._get_parent()
              return current_class
          except cls.CongressDataTypeNoParent:
              return None

      @classmethod
      def convert_to_ancestor(cls, value, ancestor_type):
          '''Convert this type's exchange value
             to ancestor_type's exchange value

          Generally there is no actual conversion because descendant type value
          is directly interpretable as ancestor type value. The only exception
          is the conversion from non-string descendents to string. This
          conversion is needed by Agnostic engine does not support boolean.

          .. warning:: undefined behavior if ancestor_type is not
                       an ancestor of this type.
          '''
          if ancestor_type == CongressStr:
              return json.dumps(value)
          else:
              return value

      @classmethod
      def _get_parent(cls):
          congress_parents = [parent for parent in cls.__bases__
                              if issubclass(parent, CongressDataType)]
          if len(congress_parents) == 1:
              return congress_parents[0]
          elif len(congress_parents) == 0:
              raise cls.CongressDataTypeNoParent(
                  'No parent type found for {0}'.format(cls))
          else:
              raise cls.CongressDataTypeHierarchyError(
                  'More than one parent type found for {0}: {1}'
                      .format(cls, congress_parents))

      class CongressDataTypeNoParent(TypeError):
          pass

      class CongressDataTypeHierarchyError(TypeError):
          pass

Re-imposing types
`````````````````
(Potential future work included here for reference.)

Policy engines may publish data for consumption by other policy engines.
When a policy engine (e.g., Z3) with narrower types publish to another policy
engine (e.g., Congress agnostic) with
unfixed types or broader types, the handling is natural. When the reverse
situation happens, the receiving policy engine may be unable to handle the
data.

As future work, the publishing policy engine can optionally infer and
re-impose narrower types where possible.

Also as future work, policy engines that require narrower types can employ
an additional import construct that imposes narrower types on the tables they
subscribe to from other policy engines.


Alternatives
------------

In this section we discuss a few ways alternative proposals may deviate from
this spec.

#. Require that all policy engines support untyped data.

   This alternative would rule out many top candidates for policy engines
   (e.g., SQL-based, influxDB-based, Z3-based).

#. Use a fixed set of types all data sources and all policy engines use.

   A fixed set of types should be a first candidate because it is the simplest
   approach. Unfortunately, it is impossible to anticipate all the types
   needed. Whenever a new policy engine is added, distinctions that may not
   have mattered before becomes critical. For example, without an SMT-based
   engine such as Z3, string enum types are not so important to distinguish
   from general string types. So when a Z3 engine is introduced, we then need
   to 1) create new types, 2) update data source drivers to specify using the
   new types, and 3) update the existing policy engines to handle the new
   types.

   The decoupled and extensible approach proposed in this spec provides for a
   much smoother way to evolve the types. Each data source driver specifies
   types as narrowly as practical, without regard to what the policy engines
   want. Each policy engine can then make use of the type information as much
   or as little as needed to map data to its own internal representations.

   So when new types, new data source drivers, or new policy engines are
   introduced, none of the existing code need to change.

#. Instead of specifying types in the data publisher, let types be imposed
   exclusively in the data consumer.

   This alternative is helpful when a fixed data type cannot be obtained from
   the data publisher (see point 5 in the section above). When a data
   source/publisher does have information on a fixed data type (inferred from
   fixed data types defined in code), it is better to avoid the unnecessary
   operational burden of the operator having to define data types at the point
   of consumption.

#. Instead of requiring that each value of a child type be directly
   interpretable as a value of a parent type, allowing a conversion between
   child and parent can give more flexibility to choose the exchange
   representation that best fits a particular type, independently of
   the type's parent type. For example, an IP address can be represented
   as a single integer value instead of as a string.

   The increase flexibility could be useful, but so far has not proven to be
   necessary.


Policy
------

Not applicable.


Policy actions
--------------

Not applicable.


Data sources
------------

Each data source driver may optionally specify data types in its value
translator. Data source drivers can be updated with explicit types as needed
by new use cases and new policy engines.

See section `Specification of data types`_ for additional details.


Data model impact
-----------------

No impact on Congress's data model as defined in congress/db/


REST API impact
---------------

No impact on REST API.


Security impact
---------------

There is little new security impact. Because new/custom types include
new/custom data handling methods, it does theoretically increase the attack
surface. Care needs to be taken to make sure the data handling methods are safe
against malformed or possibility malicious input. There are well-known best
practices to minimize such risks.

Notifications impact
--------------------

No impact.

Other end user impact
---------------------

No immediate impact on end users. This spec can indirectly benefits end
users through the additional policy engines made possible.

All existing behaviors and workflows are preserved.

Performance impact
------------------

The data and memory impact is expected to be minimal because the data is stored
and transmitted in primitive form without the overhead of custom objects.

The other main performance impact is the treatment of a piece of data as an
ancestor type.
Identifying the least ancestor type by tracing up the type hierarchy. Done
on a per-column basis, this step has negligible performance cost because the
type hierarchy is shallow and because each type has at most one ancestor. The
tracing up has worst case running time linear in the depth of the type
hierarchy.
(See `Converting to ancestor types`_ for sample
implementation.)

Other deployer impact
---------------------

No direct impact on deployers. This spec can indirectly benefits end
users through the additional policy engines made possible.

All existing behaviors and workflows are preserved.

Developer impact
----------------

No significant changes are forced on a developer. Use of type information by
a policy engine is optional. Specification of type information in a data source
driver is also optional.


Implementation
==============

Assignee(s)
-----------

Individual tasks will be picked up and tracked on launchpad. Primary contact is
ekcs <ekcs.openstack@gmail.com>.

Work items
----------

- Minimal implementation.

  - Abstract base class for type

  - Primitive types

  - Data Service Engine (DSE) changes to store and convey (optional) type
    information on columns.

- Further implementation. These further implementations are optional and can be
  done in an incremental fashion as needed by use cases.

  - Adding type information to data source drivers.

  - Standard policy engine (Agnostic) changes to use type information.

  - Extended types (e.g., IP address)

  - Categories of types (e.g., fixed enumeration of strings)

  - Framework for loading custom types defined in data source drivers

  - Inference and re-imposition of types when a policy engine publishes tables

  - Subscriber-end imposition of additional type information


Dependencies
============

No notable external dependencies.


Testing
=======

The incorporation of additional type information in the data service engine
will primarily tested through unit tests in the style of the existing tests in
congress/tests/dse2/

When a data source driver is updated to include additional type information,
additional checks on the type information can be naturally added to the
existing tempest tests that compares an independently obtained service state
to the state obtained by the data source driver and translated into tables.


Documentation impact
====================

No direct impact.


References
==========

A relevant type system for a Z3-based datalog solver:

- https://github.com/Orange-OpenSource/octant/blob/master/doc/source/user/index.rst

- https://github.com/Orange-OpenSource/octant/blob/master/octant/datalog_primitives.py
