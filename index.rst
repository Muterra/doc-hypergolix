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

Features
-------------------------------------------------------------------------------

Hypergolix is marketed towards home automation and Internet of Things
development.

Outside of local RAM, objects created with Hypergolix are always encrypted. All 
of your Hypergolix account is inflated from a user ID (the hash digest of your
bootstrap object) and your Hypergolix password.

Persistent objects
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Log in to Hypergolix, and create an object in any Python session:

.. code-block:: python

    >>> import hypergolix as hgx
    >>> hgxlink = hgx.HGXLink()
    >>> obj = hgxlink.new_threadsafe(
    ...     cls = hgx.JsonProxy,
    ...     state = 'Hello world!',
    ... )
    >>> obj
    <JsonProxy to 'Hello world!' at Ghid('AdGI5by1-Ppf0ymF26R37waIZnYADnPht5rZqLSDAmD0kV1ax94Yan_9mdd93-8i89QjDeIjBZOQhZxeG5O3HO8=')>
    
Later, logged in to the same Hypergolix account, from any other Python session, 
and even on a different computer:

.. code-block:: python

    >>> import hypergolix as hgx
    >>> hgxlink = hgx.HGXLink()
    >>> obj_address = hgx.Ghid.from_str('AdGI5by1-Ppf0ymF26R37waIZnYADnPht5rZqLSDAmD0kV1ax94Yan_9mdd93-8i89QjDeIjBZOQhZxeG5O3HO8=')
    >>> obj = hgxlink.get_threadsafe(
    ...     cls = hgx.JsonProxy,
    ...     ghid = obj_address,
    ... )
    >>> obj
    <JsonProxy to 'Hello world!' at Ghid('AdGI5by1-Ppf0ymF26R37waIZnYADnPht5rZqLSDAmD0kV1ax94Yan_9mdd93-8i89QjDeIjBZOQhZxeG5O3HO8=')>
    
Using proxies allows applications to manipulate these objects like any other:

.. code-block:: python

    >>> obj += ' Welcome to Hypergolix.'
    >>> obj.hgx_push_threadsafe()
    >>> obj
    <JsonProxy to 'Hello world! Welcome to Hypergolix.' at Ghid('AdGI5by1-Ppf0ymF26R37waIZnYADnPht5rZqLSDAmD0kV1ax94Yan_9mdd93-8i89QjDeIjBZOQhZxeG5O3HO8=')>

Sharing objects
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hypergolix objects can be shared with any other Hypergolix user, through their
public key fingerprint.

**As Alice:**

.. code-block:: python

    >>> import hypergolix as hgx
    >>> hgxlink = hgx.HGXLink()
    >>> obj = hgxlink.new_threadsafe(
    ...     cls = hgx.JsonProxy,
    ...     state = 'Hello Bob!',
    ... )
    >>> obj.hgx_ghid.as_str()
    'AWFUmWQJvo3U81-hH3WgtXa9bhB9dyXf1QT0yB_l3b6XwjB-WqeN-Lz7JzkMckhDRcjCFS1EmxrcQ1OE2f0Jxh4='
    >>> bob = hgx.Ghid.from_str('AfhB1mAR7U4Uq-UiFg9zCgIIocqmxiSnRPe5orzAjPPh71ChXWRFhwl3scgAM6w-iVXdzLVYHc-MDg4Dfx5dSVE=')
    >>> obj.hgx_share_threadsafe(bob)
    
**As Bob:**

.. code-block:: python

    >>> import hypergolix as hgx
    >>> hgxlink = hgx.HGXLink()
    >>> obj_address = hgx.Ghid.from_str('AWFUmWQJvo3U81-hH3WgtXa9bhB9dyXf1QT0yB_l3b6XwjB-WqeN-Lz7JzkMckhDRcjCFS1EmxrcQ1OE2f0Jxh4=')
    >>> obj = hgxlink.get_threadsafe(
    ...     cls = hgx.JsonProxy,
    ...     ghid = obj_address,
    ... )
    >>> obj
    <JsonProxy to 'Hello Bob!' at Ghid('AWFUmWQJvo3U81-hH3WgtXa9bhB9dyXf1QT0yB_l3b6XwjB-WqeN-Lz7JzkMckhDRcjCFS1EmxrcQ1OE2f0Jxh4=')>
    
For Bob, this object is read-only, but Alice may update the object at any time, 
and the updates will propagate to Bob's application automatically:

**As Alice:**

.. code-block:: python

    >>> obj += ' Welcome to Hypergolix!'
    >>> obj.hgx_push_threadsafe()
    
**As Bob:**

.. code-block:: python

    >>> obj
    <JsonProxy to 'Hello Bob! Welcome to Hypergolix!' at Ghid('AWFUmWQJvo3U81-hH3WgtXa9bhB9dyXf1QT0yB_l3b6XwjB-WqeN-Lz7JzkMckhDRcjCFS1EmxrcQ1OE2f0Jxh4=')>

Installing and starting Hypergolix
-------------------------------------------------------------------------------

.. toctree::
    :maxdepth: 2

    setup-1-installing
    setup-2-running

Tutorials
-------------------------------------------------------------------------------

.. toctree::
    :maxdepth: 2

    tuts-1-basics
    tuts-2-sharing
    tuts-3-advanced

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
