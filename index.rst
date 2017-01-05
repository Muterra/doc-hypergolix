===============================================================================
Hypergolix
===============================================================================

Hypergolix is "programmable Dropbox". Think of it like this:

.. raw:: html

    <div class="clearfix"></div>
    <div style="width:50%; display: inline-block; min-width: 285px;">
    <div class="sidebar admonition" style="width: 100%;">
    <p class="sidebar-title">Dropbox</p>

1.  **run** local applications
2.  on different computers
3.  using **files and folders**
4.  synced across the internet

.. raw:: html

    </div></div><div style="width:50%; display: inline-block; min-width: 285px;">
    <div class="sidebar admonition" style="width: 100%;">
    <p class="sidebar-title">Hypergolix</p>

1.  **write** local applications
2.  on different computers
3.  using **programming objects**
4.  synced across the internet

.. raw:: html

    </div></div>
    <div class="clearfix"></div>

Hypergolix runs as a local background service, just like Dropbox does. Once
it's running, instead of spending time worrying about relative IP addresses,
NAT traversal, pub/sub brokers, or mutual authentication, your code can just
do this:

.. code-block:: python

    >>> import hypergolix as hgx
    >>> hgxlink = hgx.HGXLink()
    >>> alice = hgxlink.whoami
    >>> bob = hgx.Ghid.from_str('AR_2cdgIjlHpaqGa7K8CmvSksaKMIi_scApddFgHT8dZy_vW3YgoUV5T4iVvlzE2V8qsje19K33KZhyI2i0FwAk=')
    >>> obj = hgxlink.new_threadsafe(cls=hgx.JsonProxy, state='Hello world!')
    >>> obj.share_threadsafe(bob)

and Hypergolix takes care of the rest. Alice can modify her object locally, and
so long as she and Bob share a common network link (internet, LAN...), Bob will
automatically receive an update from upstream.

Hypergolix is marketed towards Internet of Things development, but it's
perfectly suitable for other applications as well. For example, the first
not-completely-toy `Hypergolix demo app
<https://github.com/Muterra/py_hypergolix_demos/tree/master/telemeter>`_ is a
remote monitoring app for home servers.

-------------------------------------------------------------------------------
Features
-------------------------------------------------------------------------------

Network-agnostic
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Both Hypergolix objects and users are hash-addressed. Hypergolix applications
don't need to worry about the network topology between endpoints; Hypergolix is
offline-first and can failover to local storage and/or LAN servers when
internet connectivity is disrupted. This happens completely transparently to
both the application and the end user.

Client-side encryption and authentication
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

All non-local operations are enforced by cryptograpy. Specifically, Hypergolix
is backed by the `Golix protocol <https://github.com/Muterra/doc-golix>`_,
with the current implementation supporting SHA-512, AES-256, RSA-4096, and
X25519, with RSA deprecation planned by early 2018.

Except in local memory, Hypergolix objects are always encrypted (including
on-disk). Authentication is verified redundantly (by both client and server),
as is integrity. Both checks can be performed offline.

Accounts are self-hosting: all user data is extracted from a special Hypergolix
bootstrap object encrypted with the user's ``scrypted`` password.

Explicit data expiration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hypergolix data is explicitly removed, and removal propagates to upstream
servers automatically. Hypergolix lifetimes are directly analogous to object
persistence in a reference-counting memory-managed programming language: each
object (Hypergolix container) is referenced by a name (a Hypergolix address),
and when all its referents pass out of scope (are explicitly dereferenced), the
object itself is garbage collected.

Open source
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hypergolix is completely open-source. Running your own local server is easy:
``python3 -m hypergolix.service start``. Here are some source code links:

+   `Hypergolix source <https://github.com/Muterra/py_hypergolix>`_
    (~36k LoC)
+   `Hypergolix event loop management <https://github.com/Muterra/py_loopa>`_
    (~4k LoC)
+   `Hypergolix daemonization <https://github.com/Muterra/py_daemoniker>`_
    (~7k LoC)
+   `Golix implementation <https://github.com/Muterra/py_golix>`_
    (~5k LoC)

Simple to integrate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hypergolix supports all major desktop platforms (OSX, Windows, Linux; mobile
support is planned in the future). Applications interact with Hypergolix
through inter-process communication, using Websockets over ``localhost``.
Hypergolix ships with Python bindings to the API (broader language support is a
very high priority) for easy integration:

.. code-block:: python

    >>> # This connects to Hypergolix
    >>> import hypergolix as hgx
    >>> hgxlink = hgx.HGXLink()
    
    >>> # This creates a new object
    >>> obj = hgxlink.new_threadsafe(
    ...     cls = hgx.JsonProxy,
    ...     state = 'Hello World!',
    ... )
    
    >>> # This updates the object
    >>> obj += ' Welcome to Hypergolix.'
    >>> obj.hgx_push_threadsafe()
    >>> obj
    <JsonProxy to 'Hello World! Welcome to Hypergolix!' at Ghid('WFUmW...')>
    
    >>> # This is the object's address, which we need to retrieve it
    >>> obj.hgx_ghid.as_str()
    'AWFUmWQJvo3U81-hH3WgtXa9bhB9dyXf1QT0yB_l3b6XwjB-WqeN-Lz7JzkMckhDRcjCFS1EmxrcQ1OE2f0Jxh4='
    
    >>> # This retrieves the object later
    >>> address = hgx.Ghid.from_str(
    ...     'AWFUmWQJvo3U81-hH3WgtXa9bhB9dyXf1QT0yB_l3b6XwjB-WqeN-Lz7JzkMckhDRcjCFS1EmxrcQ1OE2f0Jxh4=')
    >>> obj = hgxlink.get_threadsafe(cls=hgx.JsonProxy, ghid=address)
    >>> obj
    <JsonProxy to 'Hello World! Welcome to Hypergolix!' at Ghid('WFUmW...')>

-------------------------------------------------------------------------------
Installing and starting Hypergolix
-------------------------------------------------------------------------------

.. toctree::
    :maxdepth: 2

    setup-1-installing
    setup-2-running

-------------------------------------------------------------------------------
Tutorials
-------------------------------------------------------------------------------

.. toctree::
    :maxdepth: 2

    tuts-1-basics
    tuts-2-sharing
    tuts-3-advanced

-------------------------------------------------------------------------------
API reference
-------------------------------------------------------------------------------

.. toctree::
    :maxdepth: 2
    
    api-ghid
    api-hgxlink
    api-obj
    api-proxy
    api-serialization

..
    Comment all of this stuff out until it's deemed useful
    
    Indices and tables
    -------------------------------------------------------------------------------

    * :ref:`genindex`
    * :ref:`modindex`
    * :ref:`search`
