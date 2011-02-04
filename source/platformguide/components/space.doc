.. Sirikata Documentation
   Copyright 2011, Ewen Cheslack-Postava.
   CC-BY, see LICENSE file for details.

.. _platform-components-space:

Space
=====

The space provides the actual world: the "matter" (objects) connect to
it and it provides the coordinate system, simulates global effects
like physics, and enables interaction between objects.

From the perspective of objects
connected to the space (or their object hosts) a space provides these
logical service:

* Presence (also known as space membership, session management,
  authentication)
* Coordinate System
* Physical Simulation
* Object Discovery
* Interobject Communication

This image shows the software components of the space at a high level.

.. image:: images/space.*

We'll cover each logical service and map it to the software components
in the image. This mapping should guide you to the right classes and
interfaces to look at when addressing a particular aspect of the
system.

In the image, the ``Server`` is shown as encompassing all other
components but has a dashed border. This shows that the ``Server`` has
references to all these objects but doesn't create or own them all --
it ties them together and helps coordinate them. The ``Server`` can be
thought of as the core service of the space server.


Presence
--------

When an object wants to enter the world, the space acts as the guardian. The
policy for entrance is pluggable, but most of the machinery is the same
regardless of the authentication method -- customization is provided simply by
implementing different identify verification methods.  In the code,
authentication and presence management is driven by the ``Server`` class,
supported by the ``ObjectHostConnectionManager`` and ``ObjectConnection``
classes.  The ``Authenticator`` interface abstracts authentication methods.

In many existing systems, especially games, authentication is only required for
clients: a distinction is made between objects in the world and avatars. In
Sirikata, all objects require authentication. This may sound costly, but the
cost can easily be mitigated by using intelligent ``Authenticator``
implementations.

Part of acquiring a presence in a world is providing the space with all the
basic information necessary to represent the object in the world. In this
respect, the code for presence management interacts with all other components
since it provides initial values for them. This explains why it is managed in
the central ``Server`` class, which ties together all the components of the
space server.


Coordinate System
-----------------

The unique aspect of virtual worlds is that objects inherit a shared, 3D space.
The space provides the coordinate system, controls the extents of the world, and
determines whether object's request positions can be set.  "Location" in
Sirikata refers to a collection of geometric properties in addition to position:
velocity, orientation, rotational velocity, bounds, and mesh URL.


Coordinate Segmentation
^^^^^^^^^^^^^^^^^^^^^^^

The ``CoordinateSegmentation`` is an internal service for spaces which divides
the world into regions and maps each region to a space server for simulation. It
can map from a position to the space server managing it as well as from a space
server to the region managed by it.  This is used by many other parts of the
system since it lets them discover other servers they might need to communicate
with.


LocationService
^^^^^^^^^^^^^^^

The ``LocationService`` is the externally visible service which maintains object
locations.  A ``LocationService`` implementation is essentially a large table
storing information for each object. As necessary, this information is
replicated to the ``LocationService`` on other space servers.

``LocationService`` also provides a subscription service: objects can be
subscribed for updates about other objects positions (currently performed only
by ``Proximity``). Instead of always sending all updates to all listeners,
``LocationUpdatePolicy`` determines when updates should be sent to objects. For
instance, the update policy might reduce the rate of updates when the messaging
system is overloaded or send updates more frequently to nearby objects since
changes will be more significant to them.


Physical Simulation
-------------------

Physical simulation is just a set of constraints on the values stored in
``LocationService``: requests to change position must be checked for validity
and a continuous simulation must be run to update those values according to the
laws of physics for the world.  This is shown in the figure as the Physics
component intercepting requests from objects before they reach the location
service.


Object Discovery
----------------

Once objects are inhabiting the world, they need to be able to interact. Aside
from the physical simulation, this occurs through interobject messaging. But to
message an object, you must first know how to address it. The process of
learning object identifiers is called "object discovery."

Sirikata restricts object discovery to simple geometric queries. This enables
efficient implementation and objects can filter results further when they
receive the results. In Sirikata the query has a single parameter: a solid
angle. All with solid angle larger than this value from the perspective of the
querying object will be returned.

Because this interface is fixed and the default implementation is efficient,
unlike most other components, object discovery isn't abstracted into an
interface and an implementation via plugin. Instead, the class ``Proximity``
implements query handlers, including hanlding interaction with other servers to
return objects connected to other servers.

``Proximity`` interacts heavily with ``LocationService``, which provides all the
state required to evaluate queries. It also uses ``CoordinateSegmentation`` to
determine which other servers to communicate with.  This process is handled by
the ``PintoServerQuerier`` class.


Interobject Communication
-------------------------

Once an object has identifiers for other objects, it needs to interact
with them. It does so by sending Object Datagram Protocol (ODP)
messages. These are formatted and have semantics a lot like UDP or IP packets:
they have source and destination addresses and ports and a
payload. The system knows nothing more about the format of the packets
(unless they are destined for a space service such as
``LocationService`` or ``Proximity``). Packets are best-effort: they
may be dropped if the space servers are overwhelmed with
traffic.

The two components responsible for getting messages to their
destination are the ``ObjectSegmentation`` and the ``Forwarder``.


ObjectSegmentation
^^^^^^^^^^^^^^^^^^

In order to deliver a packet, the space server needs to know *where*
to deliver it.  The first step is to lookup which space server the
destination object resides on.  The ``ObjectSegmentation`` class
performs this lookup, and externally looks like an asynchronous
key-value store. In order to avoid unnecessary lookups
(``ObjectSegmentation`` storage may be on another node, requiring a
network hop), ``OSegCache`` caches previously retrieved entries.  To
avoid a backlog, ``OSegLookupQueue`` manages the total number of
outstanding queries, and also allows coalescing of requests for the
same destination object's data.


Forwarder
^^^^^^^^^

With the destination server in hand, forwarding begins.  This is handled by the
``Forwarder`` class.  A supporting class ``LocalForwarder`` handles checking for
and forwarding to local objects. Otherwise, at its simplest, the ``Forwarer``
just ships the packet to the appropriate server, using the ``ServerIDMap`` to
convert from a server's unique identifier to an IP address.

In practice, the ``Forwarder`` is much more complicated: it has to ensure
competing flows between objects don't deny service to other flows, also manages
communication between space servers and balances it with interobject
communication, and make sure messages destined for space server components make
it to their appropriate destination. Additional supporting classes include
``ODPFlowScheduler`` and ``ForwarderServiceQueue``.


Transport Protocols
^^^^^^^^^^^^^^^^^^^

For users, it can be very inconvenient to work with best-effort datagrams. Much
of the time, they'd prefer a reliable protocol, possibly stream oriented. The
core system provides one such protocol based on Structured Streams (SST).  The
templated ``Stream`` class (and templated ``Connection`` class for support)
implement this abstraction. This protocol is used between the space server and
object host for services that need reliability or ordering. For instance,
because ``Proximity`` results contain deltas, ordering is important. These
updates are sent over SST.


Extensions
----------

These are the core set of services provided, but one could extend the space to
provide additional services.  One example where this might make sense is audio
mixing: the space server can more efficiently mix audio for each client as well
as collaborate with nearby space servers to generate a more complete mix than a
client might be able to.

If you'd like to add extensions like this, you should structure them as a
generic interface which fits into the architecture. A dummy implementation
(plugin) should allow spaces to run without the service and your implementation
would provide the actual service.  Note that usually adding an extension
requires modifying both the space and the object host.
