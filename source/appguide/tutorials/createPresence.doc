.. Sirikata Documentation
   Copyright 2011, Ewen Cheslack-Postava.
   CC-BY, see LICENSE file for details.

.. _createpres:

Creating presences
===================

The previous tutorials explained how to move an entity's presence.
This tutorial describes how to create new presences for an entity.


The create presence call
------------------------
All a scripter does to create a new entity is to execute the
``createPresence`` call.

``createPresence`` requires two arguments: a uri for the mesh to load
for the newly created presence (as a string) and a callback function
to execute once the presence connects.

The ``createPresence`` call is *non-blocking*.  What this means is
that a presence is not created as soon as the call returns.  Instead,
a presence is connected to the world and appears some time (usually
less than one second) after the call has been made.  When the presence
does get connected, the system automatically executes the callback,
passing in the newly-created presence as an argument to this callback.


Let's combine this information with the previous example on moving
presences to create a new presence, and then have it move forever in
one direction::

        function presCreatedCallback(presCreated)
        {
            //the new presence's velocity is set to
            //1 m/s in the x direction.
            presCreated.setVelocity(<1,0,0>);
        }

        //the new presence will have the mesh of a kiosk.        
        var newPresMesh = 'meerkat:///kittyvision/kiosk/flyers.dae/optimized/0/flyers.dae';
        system.createPresence(newPresMesh,presCreatedCallback);

        


Managing presences:  ``system.presences`` and ``system.self``
-------------------------------------------------------------
As you can imagine, an entity with very many presences can become
difficult to manage.  To address this, Emerson automatically manages
two global variables: ``system.presences`` and ``system.self``.  Any
time a new presence is created, it gets loaded into
``system.presences``, an array that contains all the presences
connected to an entity.  If, for instance, a scripter wanted to set
the velocity for all of his/her presences, he/she could enter the
following code::

        for (var s in system.presences)
        {
            system.presences[s].setVelocity(<1,0,0>);
        }


While, ``system.presences`` allows a scripter to track *all* the
presences he/she has connected, ``system.self`` is intended to
automatically track the most "relevant" presence for an event.

Emerson is event-based, and the majority of events in the virtual
world are associated with a single presence.  ``system.self`` gets set
to the presence associated with that event.  This may seem a little
confusing, and may make more sense through a series of examples:

 * If an entity's presence receives a message, ``system.self`` is automatically set to the presence that received the message.

 * If an entity receives a timeout event, ``system.self`` is automatically set to the presence that originally set the timeout.

 * If an entity receives a presence connected event, ``system.self`` is automatically set to the presence that was connected.

 
To provide an example of ``system.self`` in action, recall our earlier
task of creating a new presence and setting its velocity to be 1 m/s.
We can re-write ``presCreatedCallback`` to use ``system.self`` instead
of ``presCreated``::

        function presCreatedCallback(presCreated)
        {
            //the new presence's velocity is set to
            //1 m/s in the x direction.
            system.self.setVelocity(<1,0,0>);  //now using system.self
        }


       


