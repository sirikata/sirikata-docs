.. Sirikata Documentation
   Copyright 2011, Ewen Cheslack-Postava.
   CC-BY, see LICENSE file for details.

.. _gui:

Adding Graphical User Interfaces
================================

Sometimes 2D user interface widgets are a better way to interact with
a user than 3D interactions. For example, you might use a 2D widget to
request that a user execute a script locally in a sandbox or provide
a simpler interface to some interaction, like completing a purchase in
a store.

In Sirikata, 2D UIs are presented via small Javascript and HTML based
widgets. You can find examples in the UI you get with the default
client: both the scripting window and chat use these simple widgets to
allow the user to take action in the world through a 2D interface.

Components
----------
There are 3 components involved in making a UI that can perform
actions in the world.

 * The graphics object -- manages the UI widget and is required to
   allocate a widget.
 * The UI widget code -- Javascript and embedded HTML, just like the
   code in a web page.
 * Emerson driver code -- requests allocation of the UI widget and
   mediates interaction between the UI and the world, sending and
   receiving events in both directions.

Generally you will be adding the UI to an avatar which already has the
graphics object allocated. Your driver code is loaded somehow (as part
of the user's default avatar script, or dynamically by sending a
request to the user), asks the graphics object to allocate the UI
widget, and then interacts with it, possibly sending events to the UI
and receiving events back.

In this tutorial, we'll construct a simple UI with a single button
that sets your mesh to a predefined value when clicked. This is a
simple but complete example of constructing a 2D UI.

Graphics Object
---------------
In order to load a UI, you need a graphics object to allocate it
within.  The graphics object manages everything that occurs within the
window that displays the world.

In the current default avatar script
(``js/scripts/std/defaultAvatar.em``) you can find it in the global
variable ``simulator`` and it is an instance of the
``std.graphics.DefaultGraphics`` class.  Because you need the graphics
and GUI systems to be completely initialized before executing your
code, we'll place our code inside the callback it provides: ::

    simulator = new std.graphics.DefaultGraphics(
        system.self, 'ogregraphics',
        function() {
            // Invoked when DefaultGraphics is done initializing,
            // we'll put our new code in here
        }
    );

Emerson Driver Code and Allocating a UI Widget
----------------------------------------------
On disk, a UI widget is just a Javascript file that lives somewhere
under ``ogre/data/``). We'll get to the exact code later, but let's
assume we have the code in the files ``setmesh.js``. To load the GUI
module, we call the ``addGUIModule`` method on the graphics object to load
the UI::

    simulator.addGUIModule(
        "SetMesh", "setmesh.js",
        function(gui) {
            // gui is the new GUI object
        }
    );

The third parameter is a callback function. UI loading is asynchronous
because it might involve downloading resources. To avoid blocking for
a long time, a callback is invoked and provides you with the newly
created GUI object only after it is safe to start interacting with
it.  A common pattern is to store the resulting GUI object in a field
of the object that created it: ::

    simulator.addGUIModule(
        "SetMesh", "setmesh.js",
        std.core.bind(
            function(gui) {
                this._gui = gui;
            },
            this
        )
    );

In this approach, we use ``std.core.bind`` to make the ``this`` object
the same in the callback as it is for the constructor.  Any additional
initialization for the GUI, such as calling an initialization function
in the GUI with parameters from your Emerson script, should also occur
in this callback function.

Next, we know that our UI is going to generate some events we need to
handle. In particular, it is going to send a request to change mesh.
Events from the UI are specified by a string and a series of
Javascript arguments (basic types such as strings and numbers
only). To handle this event, we'll *bind* a handler for the event
``"setmesh"``. Extending our previous code, we get: ::

    simulator.addGUIModule(
        "SetMesh", "setmesh.js",
        std.core.bind(
            function(gui) {
                this._gui = gui;

                var setmesh_pres = system.self;
                handleSetMesh = function(meshurl) {
                    setmesh_pres.mesh = meshurl;
                };
                this._gui.bind("setmesh", handleSetMesh);
            },
            this
        )
    );

And that's it. When the UI sends a setmesh event, handleSetMesh will
be invoked. It uses the reference ``setmesh_pres``, the presence active
when the GUI was loaded, to set the mesh. Because we know the
arguments (and types!) that will be provided, we put a named argument
for ``meshurl``. However, if we don't know the exact form or number of
arguments, we can use the ``arguments`` keyword to extract them
dynamically.

Finally, we have all the code we need to setup the GUI from the
Emerson script. You can replace a line like ::

    simulator = new std.graphics.DefaultGraphics(system.self, 'ogregraphics');

in ``std/defaultAvatar.em`` with our updated code ::

    simulator = new std.graphics.DefaultGraphics(
        system.self, 'ogregraphics',
        function() {

            simulator.addGUIModule(
                "SetMesh", "setmesh.js",
                std.core.bind(
                    function(gui) {
                        this._gui = gui;

                        var setmesh_pres = system.self;
                        handleSetMesh = function(meshurl) {
                            setmesh_pres.mesh = meshurl;
                        };
                        this._gui.bind("setmesh", handleSetMesh);
                    },
                    this
                 )
            );
        }
    );

UI Widget Code
--------------
As discussed earlier, the UI code lives in a single Javascript
file. The UI code is similar to Emerson code: it will be fully
executed as soon as it is loaded. To continue taking actions, it sets
up HTML elements that trigger events or uses timers to periodically
take actions.

.. sidebar:: JQuery and JQuery UI

   JQuery and JQuery UI are already included into the page your
   interface is embedded in, so you can use them without including
   them yourself. They provide a lot of useful primitives for creating
   and manipulating UIs, especially if you are not familiar with web
   programming.

Here's our entire script, ``setmesh.js``::

    sirikata.ui(
        'SetMesh',
        function() {
            $('<div id="set-mesh-ui" title="SetMesh">' +             // (1)
              '  <button id="set-mesh-button">Set Mesh!</button>' +
              '</div>').appendTo('body');

            var window = new sirikata.ui.window(                     // (2)
               "#set-mesh-ui",
               {
                  width: 100,
                  height: 'auto'
               }
            );
            window.show();

            sirikata.ui.button('#set-mesh-button').click(setMesh);   // (3)

            function setMesh() {                                     // (4)
                sirikata.event("setmesh", "meerkat:///test/cube.dae/original/0/cube.dae");
            }
        }
    );

Our entire UI script is wrapped in a ``sirikata.ui`` call, which also
takes a name for our UI. This allows Sirikata to set up some helpers
and try to isolate us from other UIs. It also automatically wraps your
script that gets rid of some quirks with the way web pages load. You
should always wrap your UI code in ``sirikata.ui``.

The first statement (1) injects our UI into the page by creating a
``div`` containing all our UI elements and appending it to ``body``. This
example was very simple, but complete examples may require a few lines
to get all the components put together.  Our UI just consists of a
single button. Note that we've labeled each element with an ID.

The second line (2) uses the ID we assigned to the ``div``,
``set-mesh-ui`` to create a window out of it using ``sirikata.ui.window``.
The second parameter is a dictionary of parameters. In this case, we
just set the size of the window.

The third line (3) sets up the action that should be taken when the
``set-mesh-button`` is clicked. We simply tell it to execute a callback
``setMesh``. Many other types of actions can be bound as callbacks --
mouse over, mouse move, click, keypresses, etc.

Of course, to invoke ``setMesh`` it must be defined. The fourth
statement (4) does just that. However, the callback can't set the mesh
itself -- it doesn't have a presence to act upon. Instead, it relays
this information back to the Emerson script. Recall that we decided to
use the string ``"setmesh"`` to filter events from the UI (other elements
may also be sending events, e.g., "chat" or "scripting").  The special
``sirikata.event`` method allows you to send events out of the browser and
back into the underlying system, getting them back to your Emerson
script. The first parameter is our filter string. The remaining
arguments are passed as the arguments to the callback registered for
this event. In this case, we specify the URL for the mesh we want to
use.

And that's it, with the UI loaded and the callback setup, clicking on
the button will trigger a message back to Emerson code, which will be
handled by the Emerson ``handleSetMesh`` method. This, in turn, results
in a request to the space to set the presence's mesh.

Manipulating Widgets from Emerson
---------------------------------
This simple example only required the UI to be setup and then listen
for events, but didn't require any further manipulation of the UI from
Emerson. To further interact or update the UI from Emerson, you use
the ``std.graphics.GUI`` object returned by ``addGUIModule``.  This
object has a few simple methods like ``show``, ``hide``, and ``bind``
(as used earlier). It also includes ``GUI.eval``, which is the generic way
of interacting with the code in the UI widget. ``GUI.eval`` takes a single
string argument containing Javascript to execute in the page. In our
example, if we instead used the function::

    var setMeshURL = "meerkat:///test/cube.dae/original/0/cube.dae"
    function setMesh() {
        sirikata.event("setmesh", setMeshURL);
    }

we could change the URL we'd request by executing the
following::

    this._gui.eval("setMeshURL = 'meerkat:///test/multimtl.dae/original/0/multimtl.dae'");

which overwrites the value of ``setMeshURL``. The next time the button
is pressed and setMesh is invoked, the new value is passed back to the
script. This approach is convoluted, but it demonstrates how
additional actions can be taken by Emerson code on the GUI well after
the GUI is created.

GUI objects also provide a few methods to make the most common
interactions with GUIs simpler: ``GUI.call``, ``GUI.set``, ``GUI.variable``.
``GUI.call`` invokes a method in the context of the GUI,
making sure all variables passed through are substituted for their
values and properly escaped if necessary. For example::

    var x = 7;
    var st = 'hello "Bob"';
    this._gui.call('myfunc', x, st);

is equivalent to ::

    this._gui.eval( "myfunc(7, 'hello \"Bob\"');" );

and is much simpler and more readable than the normal approach::

    this._gui.eval( 'myfunc(' + x + ',' + escape(st) + ');' );

It looks more natural, is closer to a normal function call, and is
less error-prone since escaping is performed automatically.

``GUI.set`` sets the value of a variable in the script's context. The
above example of setting the mesh URL could be written as::

    this._gui.set('setMeshURL', 'meerkat:///test/multimtl.dae/original/0/multimtl.dae');

Of course this isn't much shorter, but the ``GUI.eval`` string is
generated automatically, escaping of strings is automatic, and you can
easily substitute variables::

    this._gui.set('setMeshURL', new_mesh_url);

Sometimes you want to be able to specify variable names in the GUI's
context in other expressions, but any strings you pass in are
converted to literals.  ``GUI.variable`` solves this by producing a
special object which will be interpreted as a variable name. For
example, to use an existing JavaScript variable ``name`` when
evaluating a function we would say ::

    this._gui.call('printname', this._gui.variable('name'));

Note that because ``GUI.set`` has a fixed format for parameters, you
don't need to use ``GUI.variable`` on its first argument.

Debugging GUI Code
------------------
The ``sirikata`` object provides utilities besides the ability to
notify the owning script of events. In particular, ``sirikata.log``
allows you to log information to the console: ::

  sirikata.log('info', 'Value: ', myvalue);

This will log the message and value of ``myvalue`` to the console. The
first parameter should always be a string and indicates the *log
level*. These levels let you control the amount of information that is
reported ('fatal', 'error', 'warn' or 'warning', 'info', 'debug',
'detailed', 'insane'). You might put a lot of logging at the 'debug' level, which
is off by default, and more important messages at 'warn',
'info', or 'fatal'.

To control the level of messages reported, start the client with a
different level for the `ui` module: ::

  cppoh --moduleloglevel=ui=debug

If you need messages provided by other components that may use the
standard logging mechanisms in browsers, you can also enable these
with the ``webview`` module: ::

  cppoh --moduleloglevel=ui=debug,webview=debug

Note that this also turns on other logging from this module besides that
generated by ``console.log`` calls from scripts, so using
``sirikata.log`` is the preferred method for logging.

Another approach to debugging UI code is to develop much of it in a
normal browser and use the debugging tools there. You can't test any
code which interacts with the rest of Sirikata (e.g. the
``sirikata.event()`` calls), but this can be helpful for the initial
development of the UI and workflow.

To do this, there are just 2 steps. First, you need to force the
system to load your GUI as a builtin. In ``ogre/data/chrome/ui.html``,
find the section where a bunch of modules are loaded by ``$LAB``. Add
a line for your own file *after* all the other ``.js`` files. For
example, given the placement described earlier, ``setmesh.js`` would
be loaded as::

    .script("../setmesh.js").wait()

Next, you'll just load up ``ui.html`` directly in a browser. You
should see at least the menu system and be able to click and select
them. If your UI doesn't pop up by default, you'll need to force it to
since the Emerson code cannot initialize it for you. In this mode,
certain functions (like sirikata.log and sirikata.event) cannot
function normally. They should just end up getting logged to the
JavaScript console in the browser and ignored.
