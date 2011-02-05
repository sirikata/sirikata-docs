.. Sirikata Documentation
   Copyright 2011, Ewen Cheslack-Postava.
   CC-BY, see LICENSE file for details.

.. _platform-tutorial-plugin:

Writing a Sirikata Plugin
=========================

The primary way to extend Sirikata is to provide a new implementation of an
interface in one of the libraries. These implementations generally live in
plugins and only the plugins required to run the desired world are loaded. For
example, the interface ``ObjectSegmentation`` defines the interface for the
component that stores the mapping from objects to the servers managing
them. However, we could implement it in different ways. Sirikata already has one
implementation called ``LocalObjectSegmentation`` for spaces which will only
ever run on a single server. This implementation is trivial. It also has
another, ``CraqObjectSegmentation``, which works in the distributed case by
using a separate key-value store based on CRAQ.

Most components in the system can have different implementations. To make these
options pluggable and not require all dependencies to build the system, we
utilize a plugin-based architecture.  This tutorial will explain how to create a
new plugin to extend the system.  Obviously this approach works for any
interface, but this tutorial will focus on a new ``ObjectSegmentation``
implementation called ``TutorialObjectSegmentation``.


Skeleton Code
-------------

The best way to start a new plugin is to base it on the skeleton plugin for
``libcore``, found in ``libcore/plugins/skeleton``. The rest of this tutorial
will be based on this code, so you can start by copying it into place for your
new plugin.  It contains only a single file, ``PluginInterface.cpp``, since it
has no real interface implementation.  You will add new files for the
implementation later in this tutorial.


Adding the Plugin to the Build System
-------------------------------------

As with the skeleton code, there are some commands for adding the skeleton
plugin to the build system. This is a good starting point, especially since the
build system is complex. Translating the skeleton commands, we might
end up adding something like this to ``build/cmake/CMakeLists.txt``.

.. code-block:: cmake

   SET(LIBSPACE_PLUGIN_TUTORIAL_DIR ${LIBSPACE_PLUGIN_DIR}/tutorial)
   SET(LIBSPACE_PLUGIN_TUTORIAL_SOURCES ${LIBSPACE_PLUGIN_TUTORIAL_DIR}/PluginInterface.cpp)

   ADD_PLUGIN_TARGET(space-tutorial
                     SOURCES ${LIBSPACE_PLUGIN_TUTORIAL_SOURCES}
                     TARGET_LDFLAGS ${sirikata_LDFLAGS}
                     LIBRARIES ${SIRIKATA_CORE_LIB} ${SIRIKATA_SPACE_LIB}
                     TARGET_LIBRARIES ${SIRIKATA_CORE_LIB} ${SIRIKATA_SPACE_LIB})
   SET(PLUGIN_INSTALL_LIST ${PLUGIN_INSTALL_LIST} space-tutorial)

First, note that we've labelled it appropriately as a space plugin by
prefixing everything with ``LIBSPACE`` .  Second, we've made sure all
our variables have our plugin name in them, ``TUTORIAL`` .  The
utility macro ``ADD_PLUGIN_TARGET`` takes care of most of the
complicated work of setting up a plugin to be built. You specify
source files, any linker flags and target libraries, and the macro
takes care of the rest. The final line adds the plugin for
installation.

If this plugin has a non-standard dependency, then we should only add
the plugin if that dependency is found.  This would look like this for
a dependency called ``tutdep``

.. code-block:: cmake

   FIND_PACKAGE(tutdep)
   IF(TUTDEP_FOUND)
     ADD_PLUGIN_TARGET(space-tutorial
       ...
     )
   ENDIF(TUTDEP_FOUND)

If necessary, add the ``tutdep`` library to the ``LIBRARIES`` argument
of ``ADD_PLUGIN_TARGET``.


Implementing the Interface
--------------------------

How the interface is implemented obviously depends on the plugin you
are writing. Here, we'll present the skeleton for
`TutorialObjectSegmentation` , which is enough to explain the basic
concepts.

Following the interface for ``ObjectSegmentation`` we create a file
``TutorialObjectSegmentation.hpp`` with the following contents ::

   #ifndef _SIRIKATA_TUTORIAL_OBJECT_SEGMENTATION_HPP_
   #define _SIRIKATA_TUTORIAL_OBJECT_SEGMENTATION_HPP_

   #include <sirikata/space/ObjectSegmentation.hpp>

   namespace Sirikata {

       class TutorialObjectSegmentation : public ObjectSegmentation {
       public:
        ...
       };

   } // namespace Sirikata

   #endif //_SIRIKATA_TUTORIAL_OBJECT_SEGMENTATION_HPP_

and a ``TutorialObjectSegmentation.cpp`` with the following contents ::

   #include "TutorialObjectSegmentation.hpp"

   namespace Sirikata {

       TutorialObjectSegmentation::TutorialObjectSegmentation() {
       }

       ...

   } // namespace Sirikata


Recall that we had to specify our source files in the build
system. Having added the implementation file, we need to add it to the
build, changing our source file list:

.. code-block:: cmake

   SET(LIBSPACE_PLUGIN_TUTORIAL_SOURCES
         ${LIBSPACE_PLUGIN_TUTORIAL_DIR}/PluginInterface.cpp
         ${LIBSPACE_PLUGIN_TUTORIAL_DIR}/TutorialObjectSegmentation.cpp
      )


Exposing the Implemented Interface from the Plugin
--------------------------------------------------

The previous steps got the implementation into the dynamic library, but it isn't
exposed to the rest of the system. Sirikata enables you to expose it as an
option via a ``Factory``.  Usually, any interface which could be instantiated
from a plugin has a ``Factory`` associated with it.  The factory allows
implementations to register a constructor function which can be invoked to
create a new instance, and names that constructor with a string.  In this way,
the library, and its users, only needs to know about the generic interface, the
factory, and what strings can be used to identify implementations.


Back in the ``PluginInterface.cpp`` we add (or replace) the following code::

   namespace Sirikata {

   static ObjectSegmentation* createTutorialOSeg(
      SpaceContext* ctx, Network::IOStrand* oseg_strand,
      CoordinateSegmentation* cseg, OSegCache* cache,
      const String& args)
   {
       return new TutorialObjectSegmentation(ctx, oseg_strand, cseg, cache);
   }

   } // namespace Sirikata

   SIRIKATA_PLUGIN_EXPORT_C void init() {
       using namespace Sirikata;
       if (space_tutorial_plugin_refcount==0) {
           using std::tr1::placeholders::_1;
           using std::tr1::placeholders::_2;
           using std::tr1::placeholders::_3;
           using std::tr1::placeholders::_4;
           OSegFactory::getSingleton()
               .registerConstructor("tutorial",
                   std::tr1::bind(&createTutorialOSeg, _1, _2, _3, _4));
       }
       space_tutorial_plugin_refcount++;
   }

   SIRIKATA_PLUGIN_EXPORT_C void destroy() {
      using namespace Sirikata;
       if (space_tutorial_plugin_refcount==0) {
           OSegFactory::getSingleton().unregisterConstructor("tutorial");
       }
   }

This registers our constructor (indirectly via a wrapper function) to the
factory when the plugin is initialized, and unregisters it when the plugin is
destroyed. In this case, you can ignore the parameters to the constructor --
they are just the parameters all ``ObjectSegmentation`` implementations must
take. While the ``createTutorialOSeg`` function must match this signature, the
constructor could take additional parameters. In fact, a common pattern is to
parse arguments from the last parameter and pass them directly into the
``TutorialObjectSegmentation`` constructor so the parsing of options is isolated
from the implementation.

If you aren't familiar with it, you probably want to learn about boost::bind. It
is a generalization of function pointers which is type-safe and allows you to
curry arguments.  In this case, it is used to turn ``createTutorialOSeg`` into a
type-safe "constructor" which returns an ``ObjectSegmentation*``.

That's all that's required -- no modifications to the core code and no need to
export symbols from the plugin.

Conclusion
----------

This tutorial should have enabled you to create a new plugin for
Sirikata which adds and exposes a new interface implementation to
Sirikata.  Frequently, after a basic implementation works one of the
first needs is to control its behavior with
options. See :ref:`platform-tutorial-options` for a tutorial on adding
options to your plugin.
