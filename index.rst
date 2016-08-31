Integrating Python applications with Hypergolix
===============================================================================

What is Hypergolix?
-------------------------------------------------------------------------------

Succinctly: Hypergolix is Dropbox, but for programming objects. It emphasizes 
sharability, is geared towards IoT, and heavily protects privacy/security.

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
