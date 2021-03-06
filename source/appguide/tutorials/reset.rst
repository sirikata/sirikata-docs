.. Sirikata Documentation
   Copyright 2011, Ewen Cheslack-Postava.
   CC-BY, see LICENSE file for details.

.. _reset:

Resetting Scripts
===================

Using the in-world scripting prompt may not be the best way for you to
build your virtual world application.  This tutorial talks about:

 * Writing a script file.
 * Loading that script on an entity.
 * Resetting an entity to an initial script.


Writing a script file 
---------------------
If you are running the entire system locally, look for the "std" and
"test" scripts directories.  (If you're on the windows build, go to
the sirikata folder you unzipped, open the "share" folder, then, open
the "js" folder; if you're on Mac or Linux, then go to "liboh", then
"plugins", then "js", then "scripts".)  Once in this folder, create a
new file, and name it "myScript.em".

The script that we're going to write is going to print the text "timer
fired!" every 3 seconds.  Think about how you would do this in an in-world
command prompt.  You'd probably enter the following commands into the
prompt::

        system.import('std/core/repeatingTimer.em');

        function testCallback()
        {
            system.print("\ntimer fired!\n");
        }

        var repTimer = new std.core.RepeatingTimer(3,testCallback);

Your script should look exactly the same.  All that executing a script
does is sequentially evaluate all the commands in a script file as if
they had been individually entered at an in-world scripting terminal.  

Loading a script
----------------
By the end of the previous section, you should have your script in
'myScript.em'.  How do you get an in-world entity to execute it?
Approach a presence in the world, and open a scripting point
associated with it.  To execute the script you just wrote, all you
need to do is call import::

        system.import('myScript.em');

The first argument of the import call should be the name of your
script.  (If you had put your script in the "test" folder, then you
would have entered 'test/myScript.em' as the first argument.)

At this point, every 3 seconds, you should see, printed into your
scripting console, 'timer fired!'.  At this point, you should know how
to write a script in an external file and get it 

Resetting
---------

Let's say that you don't want to print 'timer fired!' every 3 seconds.
Instead, you want to print 'timer REALLY fired!' every 5 seconds.  You
can certainly, by hand, call `repTimer.clear()`, and then create a new
timer.  In practice, this process may be a little cumbersome.  A
better approach would be to modify myScript.em, clear the entity's
current script, and then import the now-modified myScript.em.  Emerson
makes this easy.  

Before showing you how to do this, it's worth a little explanation.
Emerson has a special "reset" call that can only be executed from your
root sandbox.  When a scripter executes this call, almost everything that
he/she scripted is wiped away.  The two things that remain:

 * The root sandbox's presences (and their associated state)
 * A special `script` string associated with each entity.

You should know about the first of these by now, so let's focus on the
second part.  A user can set the `script` string by calling::

        system.set_script(someString);

where `someString` is a string the scripter provides.  When an Emerson
entity is reset, the Emerson runtime explicitly executes whatever is
contained in that string after wiping away all other data.

A really useful thing to put into strings like this is a simple import
call.  Let's think about how that would work for our simple timer
fired script.  After making the changes to myScript.em so that the
timer fires every 5 seconds (instead of 3) and displays a different
message, enter::

        system.set_script("system.import('std/default.em'); system.import('std/shim.em'); system.import('myScript.em');");

into your in-world scripting console, then wehenever you call
`system.reset()`, the first thing that happens after reset is that
your entity re-imports myScript.em.  You can now change your script
files, and automatically incorporate those changes.  You may be a
little confused by the two imports that precede importing from
"myScript.em".  For entities that exist in the world before you bring
them up, the system automatically imports these libraries (they do
things like allowing entities in the world to respond to scripting
messages, move events, etc.).  Because you're resetting the entire
entity state, you must include these explicitly.




