.. Sirikata Documentation
   Copyright 2011, Ewen Cheslack-Postava.
   CC-BY, see LICENSE file for details.

.. _platform-guide:

Introduction for Platform Developers
====================================

Sirikata is a large system composed of a large number of independent
services and components. Adding a new feature or fixing a bug can be a
daunting task given the size of the code base. The Platform
Developer's Guide provides a roadmap to help you get started.

The guide tries to provide three things.  First, it provides a
high-level map of the components of the system and the code base. This
should let you understand the pieces of the system, how they fit
together, and which component and code you should look at to make your
change.  Second, each service and component is broken down and
described in more detail. This should provide you enough detail to dig
into the code for a component and start to make changes.  Third, a
small set of general tutorials for how to make changes to the
system. These are specific to the code base, but not specific to any
particular component. The tutorials address issues such as how to
create a plugin, how to work with the options system, and how to
handle new dependencies.


Who is this for?
----------------

This document is targetted at developers who will be working on the
Sirikata code, rather than just using it's components to deploy a
world.  If you need to fix a bug in the system or want to add a new
plugin (e.g. a new scripting language), then you should be reading
this guide. If you want to deploy your own world or learn about
scripting objects, you should start back at the :ref:`welcome`
document.

The guide is oriented towards the primary C++ code base. While some of
the content (for instance, the :ref:`platform-components`) will be helpful
regardless of the code base you are working on, specifics about
developing for other implementations should be found in their
respective guides.


Getting Started
---------------

As mentioned above, you probably want to start with one of the three
main sections:

* :ref:`platform-tour` - A technical tour of the system and the code
  repositories.
* :ref:`platform-components` - A breakdown of the services and components of
  the system, with details of their operation.
* :ref:`platform-tutorials` - A series of tutorials describing certain aspects
  of the system which aren't specific to any component but which any
  developer will encounter when working with the system.
