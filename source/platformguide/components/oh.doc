.. Sirikata Documentation
   Copyright 2011, Ewen Cheslack-Postava.
   CC-BY, see LICENSE file for details.

.. _platform-components-oh:

Object Host
-----------

The object host handles simulation of objects and provides utilities
or them to interact with the space, other objects, and the CDN. It is
where object scripts (or behaviors) are loaded and run.

The object host has 3 main duties:

* Object simulation, including running object scripts, performing any local
  physical simulation that may be necessary
* Session management for objects, including communication for basic services
  such as location management, proximity queries, and message routing.
* Expose utilities to objects.  Examples might include persistent storage,
  higher level communication abstractions, timers, an inventory service,
  animation utilities, and so on.

In some sense, the choice of scripting language is just an extension
of the first and third duties.  Because we define a network protocol
which the components use to communicate, the choice of scripting
language is not fixed -- as long as the language can encode our
messages (or connect to our library which can encode the core set of
messages), it can work with the rest of the system.  Scripts written
in a convenient scripting language such as Python, Lua, or Ruby are
just one way to extend objects.  The object host may also provide
other extensions (implemented as plugins) which provide other fixed
functionality for objects via the same extension interface.  Examples
of these might be timers, an interface for web requests, or an
inventory service.

For more internal details on the object host and how to extend it, see the
oh_architecture.
