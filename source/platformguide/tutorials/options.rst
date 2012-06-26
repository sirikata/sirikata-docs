.. Sirikata Documentation
   Copyright 2011, Ewen Cheslack-Postava.
   CC-BY, see LICENSE file for details.

.. _platform-tutorial-options:

Adding Options using Sirikata's Option Library
==============================================

The primary method of customizing Sirikata's behavior is through
plugins: different implementations of a particular service or
interface are stored in and loaded from different plugins. To further
customize behavior, Sirikata has an options system.

Option Parsing
--------------

Options are generally specified in two places: the command line and
configuration files. Command line options are specified in the
standard format ::

   sirikata-binary --config.option.first=value1 --config.option.second=value2

Note that if you need to include special characters in an option
(spaces, ``\``, ``"``, etc), you'll need to quote and escape them: ::

   sirikata-binary "--config.option.first=value with spaces"

Configuration files are loaded by specifying them in the command line
opion ``--cfg=file.cfg``. They are formatted as a list of option-value
pairs ::

   config.option.first=value1
   config.option.second=value2

Recursive Options
-----------------

A key property of Sirikata's option system is that it will not allow
unknown options. In order to accomplish this when there are options
being added dynamically (e.g. by plugins) it uses recursive options.

For instance, consider a the interface ``LocationService`` and two
implementations, ``LocalLocationService`` and
``DistributedLocationService``. The ``LocalLocationService`` may have
no special options, but ``DistributedLocationService`` might need to
allocate a network socket to communicate with neighboring servers. In
this case, it needs a ``--port`` option.  We can't add the option
globally because the different plugins need different options and we
don't want implementation details of the plugin to leak into the main
binary using them. (Further the option name could easily conflict with
the name used by many other plugins).

Instead, we define only two options: ``loc`` and ``loc-options``.  The
first specifies which plugin we use (e.g. ``local`` or
``distributed``) and the second is just a string which gets passed
along to the plugin as options. The plugin is then responsible for
parsing them.  Of course the plugin may parse them however it likes
(it just receives them as a String), but using the Sirikata options
system is highly encouraged  to simplify the implementation,
avoid redundancy, and for consistency.

Note that these recursive options often require quoting: ::

   sirikata-binary --loc=distributed "--loc-options=--port=1234 --max-dist=1000"

Options Basics
--------------

Options are always provided initially as Strings, read from either the
command line or a configuration file.  The option system takes care of
parsing these options, verifying their formatting, and converting them
to the desired type.

This entire process should be transparent to you: you just need to specify a set
of **options**, with their types, defaults, and descriptions, which you store in
a **module**.  You then request that the options are parsed and extract and
convert the actual option values.

In the following example, we'll create options for a simple ``comm``
plugin. First, we need the options library itself: ::

   #include <sirikata/core/options/Options.hpp>

Next, we need to specify our set of options. We'll create them under the module
``"comm"`` and add two options, `host` and `port`: ::

   Sirikata::InitializeClassOptions ico("comm",NULL,
       new Sirikata::OptionValue("host", "127.0.0.1", Sirikata::OptionValueType<String>(), "The host to connect to."),
       new Sirikata::OptionValue("port", "7777", Sirikata::OptionValueType<uint16>(), "The port to connect to."),
       NULL);

Make sure that your module is unique: while a conflict isn't a problem
so long as types match up, you could allow options that shouldn't be
permitted due to somebody else's options being permitted along with yours.

``InitializeClassOptions`` can accept an arbitrarily long list of
``OptionValue``'s, terminated by a ``NULL``. This registration process should be
performed once. Usually this means registering them during plugin
initialization.  (If you need per-instance options, then you should generate a
module for each of instance. Usually this is only necessary when you could be
creating multiple instances simultaneously.)

Next, when you actually have some options to parse (for instance, when
a new instance of your implementation is requested), you can retrieve
the ``OptionSet``, parse, and extract options: ::

    OptionSet* optionsSet = OptionSet::getOptions("comm",NULL);
    optionsSet->parse(args);

    String server_host = optionsSet->referenceOption("host")->as<String>();
    uint16 server_port = optionsSet->referenceOption("port")->as<uint16>();

Note that the types requested during conversion match those specified
in the registration. The system will throw an exception if you try to
convert to the wrong type.


Adding Parsing Methods For a New Option Type
--------------------------------------------

Options are not limited to primitive types like integers and
Strings. Any type can be used as an option so long as the option
system can convert it to and from a String. For instance, Vector3
frequently appears as an option type for specifying properties like
locations and bounding regions.

Suppose you have a new type, ``MyStruct``, which you need to use as an option
value. While the Option system provides overloads for many types, you need to
make sure the template instantiation ``OptionValueType<MyStruct>`` will be valid in
order to use it as an option. In order for this to be valid, there must be an
accessible implementation of the "stream" operators (``<<`` and ``>>``). If
you're familiar with ``boost::lexical_cast``, the requirements are essentially
identical.

If we have the following definition of ``MyStruct``: ::

  struct MyStruct {
    int32 x;
    int32 y;
  };

then making the following globally accessible is a simple way to provide the
necessary conversions: ::

   inline std::ostream& operator<<(std::ostream& os, const MyStruct& rhs) {
       os << '<' << rhs.x << ',' << rhs.y << '>';
       return os;
   }

   inline std::istream& operator>>(std::istream& is, MyStruct& rhs) {
       char dummy;
       is >> dummy >> rhs.x >> dummy >> rhs.y >> dummy;
       return is;
   }

Of course the parsing could be more robust, but this minimal example is
sufficient to get conversions sufficient to use ``MyStruct`` as an option value
type.
