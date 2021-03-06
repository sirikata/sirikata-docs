.. Sirikata Documentation
   Copyright 2011, Ewen Cheslack-Postava.
   CC-BY, see LICENSE file for details.

.. _platform-tutorial-app:

Structure of a Sirikata Application: Contexts, Services and Plugins
===================================================================

In order to add new components to the system, and in some cases when creating a
plugin implementing an existing interface, you'll need to understand how a
Sirikata application (like ``space`` or ``cppoh``) is structured. This document
walks you through the process of setting up a .  Although this will be unusual
-- you would only follow this process directly to add a new binary for a
completely new service -- it is useful to provide context for how components you
write will fit into an application.

Sirikata provides a basic framework for constructing applications which provides
common infrastructure, for example managing the event loop, managing the
lifetime of the application, some simple timing functions, and managing the
services running in the application.


Definitions
-----------

To begin, we start with some definitions. These should give a sense of what each
one does, but the rest of the tutorial will fill in the details.

.. glossary::

   Context
     A small, global object for the application that maintains a small amount of
     state used across the application, including a handle to the event service,
     timing information, and internally stores and manages the list of
     Services. Note that it is global in the sense that all objects
     participating in the application have access to it, but it is not a global
     variable. There could be multiple applications, and therefore Contexts,
     within a single process.

   Service
     The coarsest granularity component the Context and event system are aware
     of.  It can be added to a Context which will cause its ``start()`` and
     ``stop()`` methods to be invoked when the application is ready to run and
     needs to shut down.

   Polling Service
     A Service which needs to be invoked periodically. It specifies its target
     frequency and implements a ``poll()`` method which will be invoked at that
     frequency.

   Plugin
     A dynamically loaded library containing implementation(s) of interface(s)
     to be used by the application. Usually these implementations are added to a
     Factory and the specific implementation is selected via configuration
     options.


A Barebones Application
-----------------------

A barebones application is simple to setup: simply create a Context to run the
application and start it.  When the application finishes running, it will return
and we just need to clean up.

First, start by including the necessary headers::

   #include <sirikata/core/service/Context.hpp>
   #include <sirikata/core/network/IOServiceFactory.hpp>
   #include <sirikata/core/network/IOService.hpp>
   #include <sirikata/core/trace/Trace.hpp>

Then create the ``main`` method and create the ``Context`` , filling in a few
parameters ::

   int main(int argc, char** argv) {

       Network::IOService* ios = Network::IOServiceFactory::makeIOService();
       Network::IOStrand* mainStrand = ios->createStrand();

       Trace::Trace* trace = new Trace::Trace(trace_file);
       Time epoch = Timer::now();

       Context* ctx = new Context("OurApplication", ios, strand, trace, epoch);

The first two lines setup the basic event handling system and these objects are
stored in ``Context`` to make them available throughout the system.  The
``Trace`` and ``epoch`` values are for collecting statistics and controlling
timed applications.

At this point, the context has almost enough information to run. The ``Context``
is itself a ``Service`` that will be added to itself to get notified of the run
starting and ending.  Then we'll start the application by calling ``run``. ::

       ctx->add(ctx);
       ctx->run(1);

The parameter to the run method is how many threads to run with. Sirikata
supports running with many threads to exploit as many cores as are available on
the system.  Events will be dispatched and handled on multiple threads.  To
ensure serialization of certain event handlers, see ``IOStrand`` or use locks
directly.

Finally, when everything has stopped running and we just need to clean up and close
the main method. ::

       ctx->cleanup();
       trace->prepareShutdown();

       delete ctx;

       trace->shutdown();
       delete trace;

       delete mainStrand;
       Network::IOServiceFactory::destroyIOService(ios);

       return 0;
   }

And that's it. This should be sufficient to get a running application -- but it
won't do anything except wait for shutdown to be requested (which you can do via a
signal, hit Ctrl-C).


Loading Plugins
---------------

In order to use implementations of interfaces that are not included in the
libraries, you must load a plugin.  For example, there is not an implementation
of the ``ObjectSegmentation`` interface provided directly in the ``libspace``
library. One implementation, ``LocalObjectSegmentation`` is implemented in the
``space-local`` plugin and is used to run a single space server system where
only a local table is required for the ObjectSegmentation.

In our hypothetical application, let's assume we need an ``ObjectSegmentation``
instance and we know ahead of time that we want the
``LocalObjectSegmentation`` .  First, we'll need to add the header that brings
in the interface and its factory, ``OSegFactory`` . Because we'll need to load
the appropriate plugin, we'll also need to use the ``PluginManager`` class. ::

   #include <sirikata/space/ObjectSegmentation.hpp>
   #include <sirikata/core/util/PluginManager.hpp>

Then, after the creation of our ``Context`` , we need to first load the plugin
to make the implementation available to the ``OSegFactory`` , and then
instantiate a copy through the factory. ::

       PluginManager plugins;
       assert( ! OSegFactory::getSingleton().hasConstructor("local") );
       plugins.load("space-local");

       assert( OSegFactory::getSingleton().hasConstructor("local") );
       ObjectSegmentation* oseg = OSegFactory::getSingleton().getConstructor("local")(ctx, ctx->mainStrand);

On the last line, the parameters are not important; the key point is that we
retrieved the constructor for the ``"local"`` implementation of
``ObjectSegmentation`` . That's it -- we dynamically loaded the plugin, making
this local implementation available, and instantiated it.  The ``assert``
statements show what the call to ``PluginManager::load()`` changed: the
``"local"`` constructor became available.

Of course we'll need to clean this up along with the other objects. ::

       delete oseg;


Adding a Service
----------------

Of course, the ``ObjectSegmentation`` we created isn't connected to anything and
won't be notified of the start and end of the application. Knowing how to
connect the component depends on the particular component, but adding the
component to the application's context is easy ::

       ctx->add(oseg);

``ObjectSegmentation`` implements the ``Service`` interface, so it will now be
notified with ``start()`` and ``stop()`` method invokations.


Conclusion
----------

Now that you know the basics of a Sirikata application, you should
take a look at the ``Context`` , ``Service`` , and ``Plugin`` classes
in the API documentation to see what else they can do.  And of course
the ``main.cpp`` files for each Sirikata binary is a good place to
start: they are long but readable, will teach you some of the common
patterns, and show how these tools are integrated with others such as
the options system.
