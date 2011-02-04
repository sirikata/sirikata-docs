.. Sirikata Documentation
   Copyright 2011, Ewen Cheslack-Postava.
   CC-BY, see LICENSE file for details.

.. _ecosystem:

The Sirikata Ecosystem
======================

Sirikata isn't a monolithic piece of software. At its most basic,
Sirikata is an architecture and set of protocols that allow the
components of that architecture to interact. Implementations of these
components allow a world developer to collect and tie together all the
pieces necessary to run a world.

The :ref:`general-architecture` describes the high level components required
in a Sirikata virtual world and their basic functions. This document
lists the known implementations of the different Sirikata
components. This is a good reference for world developers: they will
select one or more options from each category to pull together a world
that suits their, and their user's, needs.

This is just a high level breakdown. For instance, the C++
implementation is plugin based and you can choose to turn features on
or off, change which implementation you use, and interact with
different external services. These options are described in more
detail in the :ref:`world-guide` and in each implementation's
documentation.

Protocol
--------

There is one source for the protocols that allow Sirikata components
to interact.  The repository is at
http://github.com/sirikata/sirikata-protocol and is referenced by all
implementations listed here.  Note that some components which support
distributed deployment (e.g. the C++ space server) have additional,
internal protocols.  Other implementations are not required or
expected to support these internal protocols. Only protocols in the
protocol repository should be used or relied on.


Space Server
------------

* C++ Space Server -- http://github.com/sirikata/sirikata -- Currently
  the only implementation. Easy setup for running a single server, but
  also supports large, distributed deployment.

Object Host
-----------

* C++ Object Host -- http://github.com/sirikata/sirikata -- Plugin
  based scripting.

  * JavaScript - embeds Google's V8 and exposes core C++ object host
    API to JavaScript.
  * Emerson - A custom, domain specific language for virtual world
    scripting. Built as an extension on the Javascript plugin.
  * .NET Languages (Mono) - embeds the Mono .NET runtime and exposes
    core C++ object host API to .NET. Depends only on the runtime and
    is therefore language agnostic.

* KataJS -- http://github.com/sirikata/katajs -- Runs in modern web
  browsers.

  * WebSockets for communicating with space servers.
  * WebGL for 3D graphics.
  * WebWorkers support for parallelism, script isolation.
  * HTML5 for UI and interaction.


Content Distribution
--------------------

So far, content distribution has been focused around HTTP based solutions.

* Web Server (e.g. Apache)

  * Trivial implementation that just serves up static files.
  * Good for centrally controlled content, e.g. not incorporating user
    generated content.
  * Currently only CDN for KataJS since access from the browser
    requires using stock HTTP requests.

* Progressive HTTP CDN -- http://github.com/sirikata/sirikata

  * Allows content to be intelligently chunked and progressively loaded.
  * Scripts on top of web server, based on HTTP using additional
    headers.
