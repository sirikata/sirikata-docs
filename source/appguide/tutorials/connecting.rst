.. Sirikata Documentation
   Copyright 2011, Ewen Cheslack-Postava.
   CC-BY, see LICENSE file for details.

.. _emerson-tutorial-connecting:

Connecting to a Sirikata World
==============================

Before you can start scripting, you need to be connected to a world.
Currently, the connection procedure isn't very user friendly -- there
isn't a UI for it -- but its also not too challenging to customize.

First, you'll need to know what server you are connecting to.  You'll
need to know both the hostname (e.g `sample.sirikata.com`) and the
port (e.g. `7777`). If you are running your own space server, these
values will be `localhost` and `7777` by default.

To run, find the object host binary, called `cppoh` (or `cppoh_d`). If
you're using a prepackaged version then it should be under
`sirikata/bin/`. If you've built from scratch, it is under
`sirikata/build/cmake/`.  Then, we just run the binary, specifying the
hostname and port: ::

  ./cppoh_d "--servermap-options=--host=sample.sirikata.com --port=7777"

Note the quotes around the one option string -- because we need to
specify both the host and port as **suboptions** to `servermap-options`,
we need to put quotes around the entire `servermap-options` string.

If everything went right, a window should have popped up and started
displaying the world.


Client Configuration
--------------------

The default scene should provide you with a configuration . If you
want to use a customized client script (for instance, providing
different features or key bindings) you need to specify.

In Sirikata, the client is not special in anyway -- it is just a
normal object that connects to and interacts with the world just as
any other object and it just happens to also display the world in a
window and handle user input.  Because of this, the program being
executed is just the normal `object host`. Generally, when running as
the client, the configuration specifies only one object -- your avatar
-- to be loaded.  To control the set of objects that are loaded, you
can specify a scene file instead of using the default of `scene.db`:
::

   ./cppoh_d --object-factory-opts=--db=my.db

Within the database file, each object can specify a `script_type` and
`script_options`. By setting these for your avatar object, you can
alter the behavior of the client. The defaults for the client are
`js`, which specifies the JavaScript/Emerson scripting plugin, and
`--init-script=std/defaultAvatar.em`, which is a script that turns on
graphics and sets up many useful default interactions such as using
the mouse for manipulating objects and key bindings for bringing up a
scripting window for objects.

Additional Configuration
------------------------

If you need to specify additional options, or you always want to
connect to the same server, it can be more convenient to create a
configuration file which specifies all your options.  A configuration
file has one option per line, formatted as: ::

  --option=optionvalue

From the example above, we might specify the following in our
configuration file: ::

  --servermap-options=--host=sample.sirikata.com --port=7777

to avoid typing it every time we want to start the client. To run with
the configuration file, you specify it as an option to the client: ::

  ./cppoh_d --cfg=my.cfg

Now, if you have many options in `my.cfg`, they will all be included
when you specify the configuration file.
