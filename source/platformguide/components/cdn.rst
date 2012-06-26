.. Sirikata Documentation
   Copyright 2011, Ewen Cheslack-Postava.
   CC-BY, see LICENSE file for details.

.. _platform-components-cdn:

Content Distribution Network
----------------------------

The content distribution network is used to store and distribute
large, static, long-lived content.  Some examples include meshes,
textures, and object scripts. Although we can take advantage of its
use in a virtual world to optimize it, the content distribution
network looks very similar to existing solutions for other
applications. We take advantage of this by leveraging existing
solutions.

The CDN supports two levels: names and content. The inputs to the CDN
are always URIs. All URIs can respond with data directly.  However,
the preferred organization is to use a hash URI for direct data
storage. This allows efficient storage, replication, and flexibility
in the source of the data (e.g. direct from the provider, via a
torrent, etc).  Then, when referring to the content, for example when
specifying a mesh, a human readable URI is used. This attaches some
semantic meaning the URL and allows changes to be made in hierarchical
resources without requiring large adjustments (for instance, changing
a texture only requires pointing the name URI at a different hash URI;
since the mesh refers to the named resource its hash doesn't change
and so the content for the mesh remains the same but now points to the
new texture).

The CDN should support additional features that are especially useful
in this context -- attaching certain types of standard metadata, range
requests, and hashes on partial data to allow for safe use of partial
data for lower levels of detail.  More information can be found in the
detailed cdn_architecture documentation.
