===============================================================================
Basic ``bytes`` interface
===============================================================================

.. note::

    This assumes familiarity with :class:`Ghid` and :class:`HGXLink` objects.

-------------------------------------------------------------------------------
Hypergolix objects
-------------------------------------------------------------------------------

.. class:: Obj(state, api_id, dynamic, private, *, hgxlink, ipc_manager,
               _legroom, ghid=None, binder=None, callback=None)

    .. versionadded:: 0.1

    The basic Hypergolix object. Create it using :meth:`HGXLink.get` or
    :meth:`HGXLink.new`; these objects **are not intended to be created
    directly.** If you create the object directly, it won't receive state
    updates from upstream.
    
    All Hypergolix objects have a unique, cryptographic-hash-based address (the 
    ``Ghid``) and a binder (roughly speaking, the public key fingerprint of the
    object's creator). They may be dynamic or static.
    
    All Hypergolix objects have a so-called "API ID" -- an arbitrary, unique, 
    implicit identifier for the structure of the object. In traditional web
    service parlance, it's somewhere between an endpoint and a schema, which
    (unfortunately) is a pretty terrible analogy.
    
    Hypergolix objects persist nonlocally until explicitly deleted through one 
    of the :meth:`delete()` methods.

    :param HGXLink hgxlink: The currently-active :class:`HGXLink` object used 
        to connect to the Hypergolix service.
    :param state: The state of the object.
    :param hgx.utils.ApiID api_id: The API ID for the object (see above).
    :param bool dynamic: A value of ``True`` will result in a dynamic object, 
        whose state may be updated. ``False`` will result in a static object 
        with immutable state.
    :param bool private: Declare the object as available to this application 
        only (as opposed to any application for the logged-in Hypergolix user).
        Setting this to ``True`` requires an :attr:`HGXLink.app_token`.
    :param Ghid ghid: The ``Ghid`` address of the object.
    :param Ghid binder: The ``Ghid`` of the object's binder.
    :returns: The ``Obj`` instance, with state declared, **but not 
        initialized with Hypergolix.**

    .. warning::

        Hypergolix objects are not intended to be created directly. Instead, 
        they should always be created through the :class:`HGXLink`, using one 
        of its :meth:`HGXLink.new()` or :meth:`HGXLink.get()` methods.
        
        Creating the objects directly will result in them being unavailable for 
        automatic updates, and forced to poll through their :meth:`sync()` 
        methods. Furthermore, their :attr:`binder` and :attr:`ghid` 
        properties will be unavailable until after the first call to 
        :meth:`push()`.

    .. code-block:: python
 
        >>> obj = hgxlink.new_threadsafe(
        ...     cls = hgx.Obj,
        ...     state = b'Hello world!'
        ... )
        >>> obj
        <Obj with state b'Hello world!' at Ghid('bf3dR')>

    .. attribute:: state

        The read-write value of the object itself. This will be serialized and 
        uploaded through Hypergolix upon any call to :meth:`push()`.
        
        .. warning::
            
            Updating ``state`` will **not** update Hypergolix. To upload 
            the change, you must explicitly call :meth:`push()`.
        
        :rtype: bytes

        .. code-block:: python

            >>> obj
            <Obj with state b'Hello world!' at Ghid('bf3dR')>
            >>> obj.state
            b'Hello world!'
            >>> # This change won't yet exist anywhere else
            >>> obj.state = b'Hello Hypergolix!'
            >>> obj
            <Obj with state b'Hello Hypergolix!' at Ghid('bf3dR')>

    .. attribute:: ghid

        The read-only address for the object.
        
        :return Ghid: read-only address.

        .. code-block:: python

            >>> obj
            <Obj with state b'Hello world!' at Ghid('bf3dR')>
            >>> obj.ghid
            Ghid(algo=1, address=b'\xb7\xf7u\x13Y\x00\xf8k\xa9\x8fw\xab\x84>\xc0m\x10\xbc\xf9\xcf\xfd\xa9\xd5\xf1w\xda\xb9S%\x14\xeb\xc0\x81\xe0\xb9%U\x9e]5\x1f\xb4\x9e\xad\x99\x8b\xde\x1fK-\x19\xa0\t\xd23}\xc4\xaa\xe2M=E\xe8\xc9')
            >>> str(obj.ghid)
            Ghid('bf3dR')

    .. attribute:: api_id

        The read-only API ID for the object.
        
        :return bytes: read-only API ID.

        .. code-block:: python

            >>> obj
            <Obj with state b'Hello world!' at Ghid('bf3dR')>
            >>> obj.api_id
            ApiID(b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x01')

    .. attribute:: private

        Whether or not the object is restricted to this application only (see 
        above). Read-only.
        
        :return bool: read-only privacy setting.

        .. code-block:: python

            >>> obj
            <Obj with state b'Hello world!' at Ghid('bf3dR')>
            >>> obj.private
            False

    .. attribute:: dynamic

        Is the object dynamic (``True``) or static (``False``)? Read-only.
        
        :return bool: read-only dynamic/static status.

        .. code-block:: python

            >>> obj
            <Obj with state b'Hello world!' at Ghid('bf3dR')>
            >>> obj.dynamic
            True

    .. attribute:: binder

        The read-only binder of the object. Roughly speaking, the public key 
        fingerprint of its creator (see above).
        
        :return Ghid: read-only binder.

        .. code-block:: python

            >>> obj
            <Obj with state b'Hello world!' at Ghid('bf3dR')>
            >>> obj.binder
            Ghid(algo=1, address=b'\xf8A\xd6`\x11\xedN\x14\xab\xe5"\x16\x0fs\n\x02\x08\xa1\xca\xa6\xc6$\xa7D\xf7\xb9\xa2\xbc\xc0\x8c\xf3\xe1\xefP\xa1]dE\x87\tw\xb1\xc8\x003\xac>\x89U\xdd\xcc\xb5X\x1d\xcf\x8c\x0e\x0e\x03\x7f\x1e]IQ')
            >>> str(obj.binder)
            Ghid('fhB1m')

    .. attribute:: callback
    
        Gets, sets, or deletes an update callback. This will be awaited every
        time the object receives an upstream update, but it will not be called
        when the application itself calls :meth:`push()`. The callback will be
        passed a single argument: the object itself. The object's :attr:`state`
        will already have been updated to the new upstream state before the
        callback is invoked.
        
        Because they are running independently of your actual application, and 
        are called by the ``HGXLink`` itself, any exceptions raised by the 
        callback will be swallowed and logged.

        :param callback: An awaitable callback.
            
        .. warning::
        
            For threadsafe or loopsafe usage, this callback must be
            appropriately wrapped using :meth:`HGXLink.wrap_threadsafe` or
            :meth:`HGXLink.wrap_loopsafe` **before** setting it as a callback.
            
        Setting the callback:

        .. code-block:: python

            >>> obj
            <Obj with state b'Hello world!' at Ghid('bf3dR')>
            >>> async def handler(obj):
            ...     print('Updated! ' + repr(obj))
            ... 
            >>> obj.callback = handler
            
        The resulting call:

        .. code-block:: python

            >>> 
            Updated! <Obj with state b'Hello Hypergolix!' at Ghid('bf3dR')>

    .. method:: __eq__(other)
    
        Compares two Hypergolix objects. The result will be ``True`` if (and
        only if) all of the following conditions are satisfied:
        
        1.  They both have a :attr:`ghid` (else, ``raise TypeError``)
        2.  The :attr:`ghid` compares equally
        3.  They both have a :attr:`state` (else, ``raise TypeError``)
        4.  The :attr:`state` compares equally
        5.  They both have a :attr:`binder` (else, ``raise TypeError``)
        6.  The :attr:`binder` compares equally

        :param other: The Hypergolix object to compare with.
        :return bool: The comparison result.
        :raises TypeError: when attempting to compare with a 
            non-Hypergolix object.

        .. code-block:: python

            >>> obj
            <Obj with state b'Hello world!' at Ghid('bf3dR')>
            >>> obj2
            <Obj with state b'Hello world!' at Ghid('WFUmW')>
            >>> obj == obj2
            False
        
    .. note::
        
        The following methods each expose three equivalent APIs: 
        
            1.  an internal API (ex: :meth:`push()`).
                
                .. warning::
                    
                    This method **must only** be awaited from within the 
                    internal  ``HGXLink`` event loop, or it may break the 
                    ``HGXLink``, and will likely fail to work.
                    
                **This method is a coroutine.** Example usage::
                    
                    await obj.push()
                
            2.  a threadsafe API, denoted by the _threadsafe suffix 
                (ex: :meth:`push_threadsafe()`). 
                
                .. warning::
                    
                    This method **must not** be called from within the internal 
                    ``HGXLink`` event loop, or it will deadlock.
                
                **This method is a standard, blocking, synchronous method.** 
                Example usage::
                
                    obj.push_threadsafe()
                
            3.  a loopsafe API, denoted by the _loopsafe suffix 
                (ex: :meth:`push_loopsafe()`). 
                
                .. warning::
                    
                    This method **must not** be awaited from within the 
                    internal ``HGXLink`` event loop, or it will deadlock.
                    
                **This method is a coroutine** that may be awaited from your 
                own external event loop. Example usage::

                    await obj.push_loopsafe()
                    
    .. method:: recast(cls)
                recast_threadsafe(cls)
                recast_loopsafe(cls)
                
        Converts between Hypergolix object types. Returns a new copy of the
        current Hypergolix object, converted to type ``cls``.

        :param cls: the ``type`` of object to recast into.
        :returns: a new version of ``obj``, in the current class.
        
        .. warning::
        
            Recasting an object renders the previous Python object inoperable 
            and dead. It will cease to receive updates from the ``HGXLink``, 
            and subsequent manipulation of the old object is likely to cause 
            bugs with the new object as well.

        .. code-block:: python

            >>> obj
            <Obj with state b'Hello world!' at Ghid('bf3dR')>
            >>> obj.recast_threadsafe(hgx.JsonObj)
            <JsonObj with state b'Hello world!' at Ghid('bf3dR')>

    .. method:: push()
                push_threadsafe()
                push_loopsafe()
    
        Notifies the Hypergolix service (through the ``HGXLink``) of updates to
        the object. Must be called explicitly for any changes to be available 
        outside of the current Python session.

        :raises hypergolix.exceptions.IPCError: if unsuccessful.
        :raises hypergolix.exceptions.LocallyImmutable: if the object is 
            static, or if the current Hypergolix user did not create it.
        :raises hypergolix.exceptions.DeadObject: if the object is unavailable,
            for example, as a result of a :meth:`discard()` call.

        .. code-block:: python

            >>> obj
            <Obj with state b'Hello world!' at Ghid('bf3dR')>
            >>> # This state is unknown everywhere except in current memory
            >>> obj.state = b'Foo'
            >>> obj.state = b'Bar'
            >>> # Hypergolix now has no record of b'Foo' ever existing.
            >>> obj.push_threadsafe()
            >>> # The new state b'Bar' is now known to Hypergolix.

    .. method:: sync()
                sync_threadsafe()
                sync_loopsafe()
    
        Manually initiates an update through Hypergolix. So long as you create 
        and retrieve objects through the ``HGXLink``, you will not need these 
        methods.

        :raises hypergolix.exceptions.IPCError: if unsuccessful.
        :raises hypergolix.exceptions.DeadObject: if the object is unavailable,
            for example, as a result of a :meth:`discard()` call.

        .. code-block:: python

            >>> obj
            <Obj with state b'Hello world!' at Ghid('bf3dR')>
            >>> obj.sync_threadsafe()

    .. method:: share(recipient)
                share_threadsafe(recipient)
                share_loopsafe(recipient)
    
        Shares the ``Obj`` instance with ``recipient``. The recipient will 
        receive a read-only copy of the object, which will automatically update 
        upon any local changes that are :meth:`push()`\ ed upstream.

        :param Ghid recipient: The public key fingerprint "identity" of the 
            entity to share with.
        :raises hypergolix.exceptions.IPCError: if immediately unsuccessful. 
        :raises hypergolix.exceptions.DeadObject: if the object is unavailable,
            for example, as a result of a :meth:`discard()` call.
        :raises hypergolix.exceptions.Unsharable: if the object is 
            :attr:`private`\ .
            
        .. note::
            
            Successful sharing does **not** imply successful receipt.
            The recipient could ignore the share, be temporarily unavailable, 
            etc.
            
        .. note::
        
            In order to actually receive the object, the recipient must have a 
            share handler defined for the :attr:`api_id` of the object.

        .. code-block:: python

            >>> obj
            <Obj with state b'Hello world!' at Ghid('bf3dR')>
            >>> bob = hgx.Ghid.from_str('AfhB1mAR7U4Uq-UiFg9zCgIIocqmxiSnRPe5orzAjPPh71ChXWRFhwl3scgAM6w-iVXdzLVYHc-MDg4Dfx5dSVE=')
            >>> obj.share_threadsafe(bob)

    .. method:: freeze()
                freeze_threadsafe()
                freeze_loopsafe()
    
        Creates a static "snapshot" of a dynamic object. This new static object 
        will be available at its own dedicated address.

        :returns: a frozen copy of the ``Obj`` (or subclass) instance. The
            class of the new instance will match the class of the original.
        :raises hypergolix.exceptions.IPCError: if unsuccessful.
        :raises hypergolix.exceptions.LocallyImmutable: if the object is 
            static.
        :raises hypergolix.exceptions.DeadObject: if the object is unavailable,
            for example, as a result of a :meth:`discard()` call.

        .. code-block:: python

            >>> obj
            <Obj with state b'Hello world!' at Ghid('bf3dR')>
            >>> obj.dynamic
            True
            >>> frozen = obj.freeze_threadsafe()
            >>> frozen
            <Obj with state b'hello world' at Ghid('RS48N')>
            >>> frozen.dynamic
            False

    .. method:: hold()
                hold_threadsafe()
                hold_loopsafe()
    
        Creates a separate binding to the object, preventing its deletion. This 
        does not necessarily prevent other applications at the 
        currently-logged-in Hypergolix user session from removing the object.

        :raises hypergolix.exceptions.IPCError: if unsuccessful.
        :raises hypergolix.exceptions.DeadObject: if the object is unavailable,
            for example, as a result of a :meth:`discard()` call.

        .. code-block:: python

            >>> obj
            <Obj with state b'Hello world!' at Ghid('bf3dR')>
            >>> obj.hold_threadsafe()

    .. method:: discard()
                discard_threadsafe()
                discard_loopsafe()
    
        Notifies the Hypergolix service that the application is no longer 
        interested in the object, but does not delete it. This renders the 
        object inoperable and dead, preventing most future operations. However, 
        a new copy of the object can still be retrieved through any of the 
        :meth:`HGXLink.get()` methods.

        :raises hypergolix.exceptions.IPCError: if unsuccessful.
        :raises hypergolix.exceptions.DeadObject: if the object is unavailable,
            for example, as a result of a :meth:`discard()` call.

        .. code-block:: python

            >>> obj
            <Obj with state b'Hello world!' at Ghid('bf3dR')>
            >>> obj.discard_threadsafe()

    .. method:: delete()
                delete_threadsafe()
                delete_loopsafe()
    
        Attempts to permanently delete the object. If successful, it will be 
        inoperable and dead. It will also be removed from Hypergolix and made 
        unavailable to other applications, as well as unavailable to any 
        recipients of an :meth:`share()` call, unless they have called 
        :meth:`hold()`.

        :raises hypergolix.exceptions.IPCError: if unsuccessful.
        :raises hypergolix.exceptions.DeadObject: if the object is unavailable,
            for example, as a result of a :meth:`discard()` call.

        .. code-block:: python

            >>> obj
            <Obj with state b'Hello world!' at Ghid('bf3dR')>
            >>> obj.delete_threadsafe()

-------------------------------------------------------------------------------
Hypergolix  proxies
-------------------------------------------------------------------------------

.. class:: Proxy(state, api_id, dynamic, private, *, hgxlink, ipc_manager,
                 _legroom, ghid=None, binder=None, callback=None)

    .. versionadded:: 0.1
    
    The Hypergolix proxy, partly inspired by ``weakref.proxy``, is a mechanism
    by which almost any existing Python object can be encapsulated within a 
    Hypergolix-aware wrapper. In every other way, the proxy behaves exactly 
    like the original object. This is accomplished by overloading the 
    ``Proxy.__getattr__()``, ``Proxy.__setattr__()``, and 
    ``Proxy.__delattr__()`` methods.
    
    Except where otherwise noted, a Hypergolix :class:`Proxy` exposes the same
    API as an :class:`Obj`, except that the Hypergolix methods are given an
    ``hgx_`` prefix to avoid namespace collisions. For example,
    :meth:`Obj.push` becomes :meth:`Proxy.hgx_push`, and so on.

    A proxy is hashable if its :attr:`hgx_ghid` is defined, but 
    unhashable otherwise. Note, however, that this hash has nothing to do with
    the proxied object. Also note that 
    ``isinstance(proxy_obj, collections.Hashable)`` will always identify an 
    :class:`Proxy` as hashable, regardless of its actual runtime behavior.

    :param HGXLink hgxlink: The currently-active :class:`HGXLink` object used 
        to connect to the Hypergolix service.
    :param state: The state of the object. For objects using the default (*ie* 
        noop) serialization, this must be ``bytes``-like. For subclasses of 
        ``Obj``, this can be anything supported by the subclass' 
        serialization strategy.
    :param bytes api_id: The API ID for the object (see above). Should be a
        ``bytes``-like object of length 64.
    :param bool dynamic: A value of ``True`` will result in a dynamic object, 
        whose state may be update. ``False`` will result in a static object 
        with immutable state.
    :param bool private: Declare the object as available to this application 
        only (as opposed to any application for the logged-in Hypergolix user).
        Setting this to ``True`` requires an :attr:`HGXLink.app_token`.
    :param Ghid ghid: The ``Ghid`` of the object. Used to instantiate a 
        preexisting object.
    :param Ghid binder: The ``Ghid`` of the object's binder. Used to 
        instantiate a preexisting object.
    :returns: The ``Obj`` instance, with state declared, **but not 
        initialized with Hypergolix.**

    .. warning::

        As with :class:`Obj` objects, :class:`Proxy` objects are not 
        intended to be created directly.
        
    .. note::
        
        Support for Python special methods (aka "magic methods", "dunder 
        methods", etc) *is* provided. However, due to implementation details in 
        Python itself, this is accomplished by explicitly passing **all** 
        possible ``__dunder__`` methods *used by Python* to the proxied object.
        
        This has the result that IDEs will present a *very* long list of 
        available methods for :class:`Proxy` objects, even if these methods 
        are not, in fact, available. **However, the built-in** ``dir()`` **command 
        should still return a list limited to the methods actually supported by 
        the proxied:proxy combination.**
        
    .. note::
    
        Proxy objects will detect other :class:`Proxy` instances and 
        subclasses, but **they will not detect** :class:`Obj` instances or 
        subclasses unless they also subclass :class:`Proxy`. This is 
        intentional behavior.
        
    .. warning::
    
        Because of how Python works, explicitly reassigning 
        :attr:`hgx_state` is the only way to reassign the value of the 
        proxied object directly. For example, this will fail, overwriting 
        the name of the object, and leaving the original unchanged::
        
            >>> obj
            <Proxy to b'Hello world!' at Ghid('bf3dR')>
            >>> obj = b'Hello Hypergolix!'
            >>> obj
            b'Hello Hypergolix!'
            
        whereas this will succeed in updating the object state::
        
            >>> obj
            <Proxy to b'Hello world!' at Ghid('bf3dR')>
            >>> obj.hgx_state = b'Hello Hypergolix!'
            >>> obj
            <Proxy to b'Hello Hypergolix!' at Ghid('bf3dR')>

    .. code-block:: python
 
        >>> obj = hgxlink.new_threadsafe(
        ...     cls = hgx.Proxy,
        ...     state = b'Hello world!'
        ... )
        >>> obj
        <Proxy to b'hello world' at Ghid('bJQMj')>
        >>> obj += b' foo'
        >>> obj 
        <Proxy to b'hello world foo' at Ghid('bJQMj')>
        >>> obj.state = b'bar'
        >>> obj
        <Proxy to b'bar' at Ghid('bJQMj')>

    .. method:: __eq__(other)
    
        Compares the ``Proxy`` with another object. The comparison 
        recognizes other Hypergolix objects, comparing them more thoroughly
        than other objects.
        
        If ``other`` is a Hypergolix object, the comparison will return
        ``True`` if and only if:
        
        1.  The :attr:`Obj.ghid` attribute compares equally
        2.  The :attr:`Obj.state` attribute compares equally
        3.  The :attr:`Obj.binder` attribute compares equally
        
        If, on the other hand, the ``other`` object is not a Hypergolix object 
        or proxy, it will directly compare ``other`` with :attr:`hgx_state`.

        :param other: The object to compare with
        :rtype: bool

        .. code-block:: python

            >>> obj
            <Proxy to b'Hello world!' at Ghid('bf3dR')>
            >>> obj2
            <Proxy to b'Hello world!' at Ghid('WFUmW')>
            >>> obj == obj2
            False
            >>> not_hgx_obj = b'Hello world!'
            >>> not_hgx_obj == obj
            True
            >>> obj2 == not_hgx_obj
            True
