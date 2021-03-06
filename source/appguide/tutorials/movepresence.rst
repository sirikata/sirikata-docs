.. Sirikata Documentation
   Copyright 2011, Ewen Cheslack-Postava.
   CC-BY, see LICENSE file for details.

.. _movepresence:

Moving in the World
===================

It would be a pretty boring world if all presences just stayed in one
place.  This tutorial explains how a scripter can move presences
throughout the world.

There are two basic types of movement in the system: translational and
rotational.  In translational movement, every point of the presence's
mesh moves the same distance; in rotational movement, a presence's
mesh spins around a fixed axis.  A useful way of understanding the
difference between translational and rotational movement is to think
about a wheeled office chair.  A person wanting to translate his/her
office chair would roll it from one point in a room to another.  A
person wanting to rotate his/her office chair would spin in place on
the office chair.

Translational and rotational movement can be combined (ie, one can use
Emerson to translate and rotate a presence simultaneously).  For the
simplicity of this tutorial, we instead focus on each separately.

Before we begin, a quick note on code in the following examples: the
Emerson run-time keeps track of all of an entity's connected presences
in an array available to the scripter as ``system.presences``.  For
the example code used in this tutorial, we'll assume that the presence
being moved is named by ``system.self``.  


Translational motion
--------------------
Emerson allows a scripter to either instantly teleport a presence to a
new position or to assign a velocity to the presence.  A presence with
an assigned velocity automatically moves through the world in the
assigned direction and rate.  For both position and velocity, Emerson
uses standard Cartesian coordinates, and represents each using
3-dimensional vectors.  We'll talk about position and velocity
separately below, but it may be useful for the reader to refer to the Vec3
type described here<lkjs> before beginning.


Teleport/SetPosition
++++++++++++++++++++
Assume that you have a presence that you want to instantly translate
from its current position to a position 100m away in the x direction.

We'll break such a script into 3 pieces:

  * Getting the presence's current x, y, and z coordinates in the space.
  * Specifying a new 3-dimensional position based on the presence's current x, y, and z coordinates in the space.
  * Requesting the system to move your object to this new position.


Here's the entire script to perform the above::

        var pos   = system.self.getPosition();
        pos.x     = pos.x + 100;
        system.self.setPosition(pos);

The above is explained more methodically below.

*Getting the presence's current coordinates in the space*

To get a 3-dimensional vector representing the current position of a
presence in space, simply use the ``getPosition()`` function.  As an
example, open a scripting window and enter::

        var pos = system.self.getPosition();
        system.print(pos);

You should see the position of a presence printed in the following
form: <x,y,z>.  You can access the individual x, y, and z fields of
this position with the accessors `.x`, `.y`, and `.z`, respectively.
That is, if you wanted to just print the x position of a presence, you
would replace the second line of the above function as shown below::

        var pos = system.self.getPosition();
        system.print(pos.x);

*Specifying a new 3-dimensional position based on ``pos``*
After the previous section, the variable ``pos`` stores
the current position associated with ``system.self``.  We need
to specify a new target position.  Doing so is simple::

        pos.x = pos.x + 100;

It is important to note, the above line does *not* move the presence
by itself.  It instead changes the value stored in ``pos``.  ``pos``
now describes the position that we want to move our presence to.

*Requesting the system to move your object to ``pos``*
To actually set the position of ``system.self`` to ``pos``,
one simply uses the ``setPosition`` function::

        system.self.setPosition(pos);

The ``setPosition`` function takes in a single argument, a
3-dimensional vector, and instantly moves the associated presence to
that point in the space.

SetVelocity
+++++++++++++++++++
The above example instantly moved a presence from one point to another.
While such an operation is possible in a virtual world, it certainly
does not agree with (most of) the real world: one wouldn't see a train
teleport from one station to another -- instead, the train acquires a
velocity and moves from point to point non-instantaneously.

Emerson provides similar capabilities for presences by allowing a
scripter to set a presence's velocity (both magnitude and direction).
(Note: in Emerson, a scripter *cannot* set higher order movement
properties for presences such as acceleration or jerk.)

Consider the simple task of moving a stationary presence 30m in the y
direction.  To do so, a scripter would use the ``setVelocity``
command.  Like the ``setPosition`` command described in the previous
section, ``setVelocity`` takes in a single argument: a 3-dimensional
vector.  The x, y, and z components of this argument 3-dimensional vector
specifies the velocity that the presence should move in meters per
second (in the x, y, and z directions respectively).  You can
experiment with the ``setPosition`` function pretty easily.  For
instance, at a scripting prompty, enter the following code::

        var zeroVelocity = new util.Vec3(0,0,0);
        var posYVelocity = new util.Vec3(0,1,0);
        var negYVelocity = new util.Vec3(0,-1,0);


Now, to get a presence to stop moving, just enter::

        system.self.setVelocity(zeroVelocity);

To get a presence to start moving in the positive y direction, enter::

        system.self.setVelocity(posYVelocity);

And, to get a presence to start moving in the negative y direction, enter::

        system.self.setVelocity(negYVelocity);


To complete the simple task of moving a stationary presence 30m in the y
direction, enter into the command prompt::

        system.self.setVelocity(posYVelocity);

and then 30 seconds later::

        system.self.setVelocity(zeroVelocity);

The first command should start the presence moving in the y direction at
1 m/s.  The second command should stop the presence.  Manually
entering such commands obviously leaves something to be desired.  For
a more robust solution, read about ``timeout``s in the tutorial.



Rotational motion
-----------------
Ugh.  I hate quaternions.
<lkjs>
