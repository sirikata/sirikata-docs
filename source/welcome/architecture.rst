.. Sirikata Documentation
   Copyright 2011, Ewen Cheslack-Postava.
   CC-BY, see LICENSE file for details.

.. _general-architecture:

Sirikata Architecture
=====================

Introduction
------------

This document describes the basic components of a Sirikata world at
the highest (and hopefully non-technical) level.  Whether you're
developing an application in a Sirikata world or creating and
deploying your own world, you'll need to understand these basic
components.  World developers need to understand these components
because they will configure and run them to pull together a virtual
world. Application developers need to understand them because they
interact with each component.

At the highest level, Sirikata is composed of three parts: the space,
the object hosts, and the content distribution network:

.. glossary::

   Object Host
     Simulates object behavior in the world by running scripts. User
     clients are object hosts that also know how to run a display.

   Space
     A collection of one or more servers that simulate the
     world. Mediates interaction between objects, such as physics and
     communication.

   Content Distribution Network
     Stores semi-permanent to permanent data and delivers it to the
     other components. For instance, meshes that are used to display
     objects are stored on a CDN and client object hosts can download
     them to view the world.

.. sidebar:: Example Deployments

   .. image:: images/deployment_open.*
      :width: 200px

   .. image:: images/deployment_game.*
      :width: 200px

   Two ways to deploy Sirikata. In the first, one company runs the
   infrastructure for the world (space), but leaves other services up
   to users and third parties. In the second, to ensure game rules are
   enforced and a good user experience, the game company runs all
   servers except for object hosts acting as clients.

All of these components are required for a functioning world. Without
object hosts, the world wouldn't have any objects in it. Without the
space, object hosts wouldn't know how to interact and other
centralized global services like physics wouldn't be
available. Without the content distribution network, there wouldn't be
a place to store and retrieve data that lets us display the world.

These components can be put together in different ways to create
different types of worlds.  The sidebar shows some examples.

The rest of this document gives a little more detail on these
components, but tries to remain non-technical. The description here
should be sufficient for application developers. World developers may
need to understand the components in more detail to customize their
world. Greater detail can be found in the
:ref:`world-architecture` of the
:ref:`world-guide`. Finally, if you're looking to extend the
Sirikata platform, check the :ref:`platform-components` in the
:ref:`platform-guide`.

Space
-----

The space is what you might think of as the world around us: it isn't
the objects but the set of rules they must follow and how they
interact with each other.  In Sirikata this boils down to only a small
set of services: the space manages objects locations, physics
simulation, lets objects learn about other nearby objects, and enables
those objects to communicate with each other.  All the rest of the
functionality of the world can be built on this very small base.

Spaces may also provide additional services. For example, a world
could function perfectly well with sounds handled entirely by
exchanges between objects. However, it might be more cost effective to
have the space manage sound to efficiently generate sound for each
object that wants to listen.

Spaces can be simulated using a single server, but are commonly
simulated across a number of *space servers*, enabling them to handle
larger regions, more objects, and more clients.


Object Host
-----------

An object host simulates objects, allowing them to specify their
behavior in scripts and providing them access to spaces and
CDNs. At their core, object hosts are simple: they embed a scripting
language for objects and expose ways for the objects to connect to
spaces and CDNs and usually provide some easier methods for
interacting with other objects.

The examples in the sidebar suggest how object hosts might be used. In
an open world, anybody running an object host can connect to the
space. The owner adds their scripted objects, which connect to that
space and start interacting with other objects. They might add code
while they are in the world to extend the behavior of objects, perhaps
teaching it how to interact with a new type of object or provide a new
service to other avatars.

In the second example, a company is developing a game. Because they
want complete control over how objects behave in the world, they run
all the object hosts containing objects in the world (from mountains,
to swords, to goblins), except for those with the players.

In both cases, clients are just a special type of object host that
knows how to display the world and respond to user input. The object
that manages this is the avatar. This means that clients and objects
in the world aren't differentiated, both can be and are scripted using
the same tools.

A graphical display isn't the only extension an object host can have.
Object host can support a variety of plugins: display of the 3D world,
input from the user, sound input and output, different scripting
languages, access to permanent storage (e.g. disk), and interoperation
with other systems (e.g. HTTP access) are just a few examples.


Content Distribution Network
----------------------------

The content distribution network stores and serves long-lived content
the world requires to run. The most obvious example is mesh data,
which often can't be distributed up-front (for instance, because users
add objects with their own meshes to the world).

A "content distribution network" can be as simple as a web server, and
in fact that is a common solution. The CDN really only needs to be
able to serve requests for files.  However, the CDN might be much more
complicated, handling a very large number of users, intelligently
serving data from servers near the source of requests, doing advanced
processing of content (like simplification, conversion to progressive
formats for streaming), and caching files to reduce the cost of
running the CDN.
