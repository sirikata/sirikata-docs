.. Sirikata Documentation
   Copyright 2011, Ewen Cheslack-Postava.
   CC-BY, see LICENSE file for details.

.. _createent:

Creating entities
===================

By now, you should know how to add presences to an entity.  This
tutorial describes how you would go about creating a **new** entity.
We'll use an example in which we create a new entity, and then have
that entity's presence move through the world.


The create entity call
----------------------
All a scripter does to create a new entity is to execute the
``create_entity`` call.  The call:

  * creates a new entity on the same entity host that processed the ``create_entity`` call and
  * automatically connects a presence for that entity to the world.

The second task requires some additional information, and requires
that the ``create_entity`` call takes several parameters.  In order,
these parameters are:
  
  * The position the entity's presence should occupy in the world (Vec3).
  * The type of script to import.  (For emerson scripts, include "js".).
  * The filename of the script to import.
  * A URI to the mesh the entity's automatically-connected presence should take.
  * A scaling factor for that mesh.
  * A steradian value for the solid angle query the entity's automatically-connected presence should issue. 

  
Example
--------
Now that you know the structure of a ``create_entity`` call, consider
the following code::

        //new entity's presence has the mesh of a sphere
        var meshToChangeTo = "meerkat:///test/sphere.dae/original/0/sphere.dae";

        //new entity's presence will be located two units away from creating
        //presence's position
        var newPos = system.self.getPosition();
        newPos.x = newPos.x + 2;

        //First thing that the new entity will do after its presence connects
        //to space is import the following file.
        var scriptToImport = "test/testMove.em";


        system.create_entity(newPos,
                     "js",    //this arg will almost always be 'js'
                     scriptToImport,
                     meshToChangeTo,
                     1.0,     //how do you want to scale the mesh of
                              //the entity's new presence
                     3        //what is the solid angle query that
                              //the entity's initial presence
                              //queries with.
                    );


This code would be executed on an already-existing entity to create a
new entity.  The new entity's automatically-connected presence would
have a mesh of ``meshToChangeTo`` (a sphere), be positioned a the
x,y,z coordinates contained in ``newPos`` (2 meters away from the
previous entity's first presence), import a script from
``scriptToImport`` (in filename 'test/testMove.em'), and set a solid
angle query of 3 steradians.



The actual structure in the script 'test/testMove.em' may be a little
different than scripters are used to.  Most of the scripts that we
have discussed to this point operated on entities that already had
presences connected to the world: scripters were able to assume
``system.self`` existed, and was connected to the world.  In
contrast, the imported script executes **before** its first presence
is automatically connected.  This means that 'test/testMove.em' must
actually catch the first presence's connection before operating on it.

This is easy, and is accomplished through the
``onPresenceConnected`` system call.  Below is the 'test/testMove.em'
script, which uses the ``onPresenceConnected`` call::

        //whenever a presence in the system connects to the world,
        //execute the argument of onPresenceConnected
        system.onPresenceConnected(moveFunc);


        function moveFunc(newPres)
        {
            //set a unit velocity for the first presence in the x direction.
            system.self.setVelocity(<1,0,0>);
        }

        
       


