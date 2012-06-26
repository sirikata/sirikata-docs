.. Sirikata Documentation
   Copyright 2011, Ewen Cheslack-Postava.
   CC-BY, see LICENSE file for details.

.. _platform-components:

Sirikata Components
===================

This section gives an in-depth description of the services of the
Sirikata system. Each service is broken down into a number of
components, each of which are also described in detail. This section
should be used as a guide for understanding what component provides
each aspect of the high-level services and how these components
interact.  This is not a code reference, although it will provide some
pointers so you can jump from the description of a component to its
code.



Sirikata provides the basic services and functionality needed by
virtual world applications.  Applications tie these services together
and build on top of them to provide a unique experience to end users.

Sirikata breaks an entire virtual world system -- including the
components executed by end users -- into three high level services:
the object hosts, the space, and the content distribution network.
These services loosely correspond to computation, communication, and
storage, respectively.

To deploy a virtual world application, the application developer will
provide a single space service (possibly distributed across many
servers) which is, in some sense, the "world."  Depending on the
application, the provider might also run object hosts, which connect
to the space and run objects provided by the world (the scenery and
bots in a game, for example), and a CDN to handle storage of large,
static data, such as geometry, textures, and prerecorded audio. As
described later, clients connect to the world by running an object
host locally which, at a minimum, simulates their avatar or camera
object.

This high level description leaves out a lot of detail, but gives an
idea of what is involved in building and deploying a virtual world
using Sirikata.  Note that this configuration isn't the only one
possible -- for instance, a CDN might not be provided and all
resources might either be directly deivered, as in a procedurally
defined world, or the provider might not provide a CDN, relying on
users to find hosting, for example via web hosting.  In this manual we
hope to describe the most common deployment, making note of
alternatives where appropriate.

With this high level context, the following describes each of these
components in a bit more detail, but focuses on each of their external
interfaces.  For details on the internal architectures of these
components, including ways in which they can be customized and
extended, see their corresponding architecture pages:

* space architecture
* oh_architecture
* cdn_architecture


.. toctree::

   components/space
   components/oh
   components/cdn
