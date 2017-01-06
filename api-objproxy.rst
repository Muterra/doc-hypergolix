===============================================================================
Basic ``bytes`` interface
===============================================================================

.. note::

    This assumes familiarity with :class:`Ghid` and :class:`HGXLink` objects.

-------------------------------------------------------------------------------
Hypergolix objects
-------------------------------------------------------------------------------

.. class:: ObjBase(hgxlink, state, api_id, dynamic, private, ghid=None, binder=None)

    .. versionadded:: 0.1

    A base class for Hypergolix-connected objects. Does not perform any 
    serialization of its state -- in other words, ``ObjBase`` instance state 
    (the actual value of the object) must be ``bytes``-like.
    
    When first created **through the ``HGXLink``,** the declared state is 
    passed to Hypergolix through a call to an :meth:`_hgx_push()` method to 
    create the object; however, ``ObjBase.__init__()`` does not do this 
    internally. Thereafter, any changes to local state must be explicitly 
    pushed upstream through an :meth:`_hgx_push()` (or equivalent) call.
    
    All Hypergolix objects have a unique, cryptographic-hash-based address (the 
    ``Ghid``) and a binder (roughly speaking, the public key fingerprint of the
    object's creator). They may be dynamic or static.
    
    All Hypergolix objects have a so-called "API ID" -- an arbitrary, unique, 
    implicit identifier for the structure of the object. It is analogous to an 
    API endpoint for traditional web services. Applications may declare custom 
    API IDs, or use the default value for the object's serialization strategy.
    
    Until explicitly shared with someone else, all Hypergolix objects are only 
    visible to the person that created them. This is enforced through the Golix
    cryptographic protocol, which protects the objects any time they leave 
    local memory. However, by default, for any particular Hypergolix user, any 
    object is available to every locally-running Hypergolix application. This 
    behavior can be overridden by the application by declaring an object to be 
    private. In the future, the Hypergolix service itself will also allow the 
    user to override this behavior at a system level.
    
    Hypergolix objects persist nonlocally until explicitly deleted through one 
    of the :meth:`_hgx_delete()` methods.

    :param HGXLink hgxlink: The currently-active :class:`HGXLink` object used 
        to connect to the Hypergolix service.
    :param state: The state of the object. For objects using the default (*ie* 
        noop) serialization, this must be ``bytes``-like. For subclasses of 
        ``ObjBase``, this can be anything supported by the subclass' 
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
    :returns: The ``ObjBase`` instance, with state declared, **but not 
        initialized with Hypergolix.**

    .. warning::

        Hypergolix objects are not intended to be created directly. Instead, 
        they should always be created through the :class:`HGXLink`, using one 
        of its :meth:`HGXLink._new()` or :meth:`HGXLink._get()` methods.
        
        Creating the objects directly will result in them being unavailable for 
        automatic updates, and forced to poll through their :meth:`hgx_sync()` 
        methods. Furthermore, their :attr:`hgx_binder` and :attr:`hgx_ghid` 
        properties will be unavailable until after the first call to 
        :meth:`_hgx_push()`.
        
    .. note::
        
        The default API ID for base objects with no serialization is::
        
            0x 0000000000000000 0000000000000000 0000000000000000 0000000000000000
               0000000000000000 0000000000000000 0000000000000000 0000000000000001
               
        or, more concisely::
       
            bytes(63) + b'\x01'

    .. code-block:: python
 
        >>> obj = hgxlink.new_threadsafe(
        ...     cls = hgx.ObjBase,
        ...     state = b'Hello world!'
        ... )
        >>> obj
        <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>

    .. attribute:: hgx_state

        The read-write value of the object itself. This will be serialized and 
        uploaded through Hypergolix upon any call to :meth:`_hgx_push()`.
        
        .. warning::
            
            Updating ``hgx_state`` will **not** update Hypergolix. To upload 
            the change, you must explicitly call :meth:`_hgx_push()`.
        
        :rtype: bytes

        .. code-block:: python

            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> obj.hgx_state
            b'Hello world!'
            >>> # This change won't yet exist anywhere else
            >>> obj.hgx_state = b'Hello Hypergolix!'
            >>> obj
            <ObjBase with state b'Hello Hypergolix!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>

    .. attribute:: hgx_ghid

        The read-only address for the object.
        
        :return Ghid: read-only address.

        .. code-block:: python

            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> obj.hgx_ghid
            Ghid(algo=1, address=b'\xb7\xf7u\x13Y\x00\xf8k\xa9\x8fw\xab\x84>\xc0m\x10\xbc\xf9\xcf\xfd\xa9\xd5\xf1w\xda\xb9S%\x14\xeb\xc0\x81\xe0\xb9%U\x9e]5\x1f\xb4\x9e\xad\x99\x8b\xde\x1fK-\x19\xa0\t\xd23}\xc4\xaa\xe2M=E\xe8\xc9')
            >>> str(obj.hgx_ghid)
            Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')

    .. attribute:: hgx_api_id

        The read-only API ID for the object.
        
        :return bytes: read-only API ID.

        .. code-block:: python

            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> obj.hgx_api_id
            b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x01'

    .. attribute:: hgx_private

        Whether or not the object is restricted to this application only (see 
        above). Read-only.
        
        :return bool: read-only privacy setting.

        .. code-block:: python

            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> obj.hgx_private
            False

    .. attribute:: hgx_dynamic

        Is the object dynamic (``True``) or static (``False``)? Read-only.
        
        :return bool: read-only dynamic/static status.

        .. code-block:: python

            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> obj.hgx_dynamic
            True

    .. attribute:: hgx_binder

        The read-only binder of the object. Roughly speaking, the public key 
        fingerprint of its creator (see above).
        
        :return Ghid: read-only binder.

        .. code-block:: python

            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> obj.hgx_binder
            Ghid(algo=1, address=b'\xf8A\xd6`\x11\xedN\x14\xab\xe5"\x16\x0fs\n\x02\x08\xa1\xca\xa6\xc6$\xa7D\xf7\xb9\xa2\xbc\xc0\x8c\xf3\xe1\xefP\xa1]dE\x87\tw\xb1\xc8\x003\xac>\x89U\xdd\xcc\xb5X\x1d\xcf\x8c\x0e\x0e\x03\x7f\x1e]IQ')
            >>> str(obj.hgx_binder)
            Ghid('AfhB1mAR7U4Uq-UiFg9zCgIIocqmxiSnRPe5orzAjPPh71ChXWRFhwl3scgAM6w-iVXdzLVYHc-MDg4Dfx5dSVE=')

    .. method:: __eq__(other)
    
        Compares the ``ObjBase`` with another ``ObjBase`` instance (or an 
        instance of one of its subclasses). The result will be ``True`` if (and
        only if) all of the following conditions are satisfied:
        
        1.  They both have an :attr:`hgx_ghid` attribute (else, 
            ``raise TypeError``)
        2.  The :attr:`hgx_ghid` attribute compares equally
        3.  They both have an :attr:`hgx_state` attribute (else, 
            ``raise TypeError``)
        4.  The :attr:`hgx_state` attribute compares equally
        5.  They both have an :attr:`hgx_binder` attribute (else, 
            ``raise TypeError``)
        6.  The :attr:`hgx_binder` attribute compares equally

        :param other: The ``ObjBase`` (or subclass) instance to compare 
            with.
        :return bool: The comparison result.
        :raises TypeError: when attempting to compare with a 
            non-``ObjBase``-like object.

        .. code-block:: python

            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> obj2
            <ObjBase with state b'Hello world!' at Ghid('AWFUmWQJvo3U81-hH3WgtXa9bhB9dyXf1QT0yB_l3b6XwjB-WqeN-Lz7JzkMckhDRcjCFS1EmxrcQ1OE2f0Jxh4=')>
            >>> obj == obj2
            False

    .. method:: _hgx_register_callback(callback)
                hgx_register_callback_threadsafe(callback)
                hgx_register_callback_loopsafe(callback, target_loop)

        Registers an update callback. This callback will be called every time 
        the object receives an upstream update (pull) from Hypergolix. It will 
        not be called when the application itself calls :meth:`_hgx_push()` (or 
        equivalent). The callback will be passed a single argument: the object 
        itself. :attr:`hgx_state` will already have been updated to the new 
        upstream state when the callback is called.
        
        Because they are running independently of your actual application, and 
        are invoked by the ``HGXLink`` itself, any exceptions raised by the 
        callback will be swallowed and logged.

        :param callback: For threadsafe callbacks, this should be a callable. 
            For the other callbacks, this should be an awaitable.
        :param target_loop: For loopsafe callbacks, the event loop to call the 
            callback within.
            
        .. note::
        
            All three of these methods are synchronous calls. They may be 
            invoked anywhere, at any time.
            
        .. warning::
        
            Any given ``ObjBase`` instance can have at most a single update 
            callback. Subsequent calls to any of the 
            :meth:`_hgx_register_callback()` methods will overwrite the 
            existing callback without warning.
            
        .. note::
            
            The :meth:`_hgx_register_callback()` callback will be awaited from 
            within the internal ``HGXLink`` event loop.
            
        .. note::
            
            The :meth:`hgx_register_callback_threadsafe()` callback will be 
            called from a dedicated, single-use, disposable thread.
            
        .. note::
            
            The :meth:`hgx_register_callback_loopsafe()` callback will be 
            called from within the passed ``target_loop``.
            
        Setting the callback:

        .. code-block:: python

            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> def handler(obj):
            ...     print('Updated! ' + repr(obj))
            ... 
            >>> obj.hgx_register_callback_threadsafe(handler)
            
        The resulting call:

        .. code-block:: python

            >>> 
            Updated! <ObjBase with state b'Hello Hypergolix!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>

    .. method:: hgx_clear_callback()
    
        Clears any existing update callback. Idempotent.

        .. code-block:: python

            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> def handler(obj):
            ...     print('Updated! ' + repr(obj))
            ... 
            >>> obj.hgx_register_callback_threadsafe(handler)
            >>> obj.hgx_clear_callback()
            >>> # Note idempotency: this does not raise.
            >>> obj.hgx_clear_callback()
        
    .. note::
        
        The following methods each expose three equivalent APIs: 
        
            1.  an internal API, denoted by a leading underscore 
                (ex: :meth:`_hgx_push()`).
                
                .. warning::
                    
                    This method **must only** be awaited from within the 
                    internal  ``HGXLink`` event loop, or it may break the 
                    ``HGXLink``, and will likely fail to work.
                    
                **This method is a coroutine.** Example usage::
                    
                    await _hgx_push()
                
            2.  a threadsafe external API, denoted by the _threadsafe suffix 
                (ex: :meth:`hgx_push_threadsafe()`). 
                
                .. warning::
                    
                    This method **must not** be called from within the internal 
                    ``HGXLink`` event loop, or it will deadlock.
                
                **This method is a standard, blocking, synchronous method.** 
                Example usage::
                
                    hgx_push_threadsafe()
                
            3.  a loopsafe external API, denoted by the _loopsafe suffix 
                (ex: :meth:`hgx_push_loopsafe()`). 
                
                .. warning::
                    
                    This method **must not** be awaited from within the 
                    internal ``HGXLink`` event loop, or it will deadlock.
                    
                **This method is a coroutine** that may be awaited from your 
                own external event loop. Example usage::

                    await hgx_push_loopsafe()
                    
    .. classmethod:: _hgx_recast(obj)
                    hgx_recast_threadsafe(obj)
                    hgx_recast_loopsafe(obj)
                
        Converts a local ``ObjBase`` (or subclass) object into a different 
        object class. Recasting can only convert between direct descendants and 
        ancestors -- ie, a :class:`JsonObj` could be converted to/from an 
        :class:`ObjBase`, but a :class:`JsonObj` cannot be converted to/from a 
        :class:`PickleObj`.
        
        Returns a new instance of the object, recast as the calling class. 

        :param obj: the :class:`ObjBase` instance to recast.
        :returns: a new version of ``obj``, in the current class.
        :raises TypeError: when attempting to recast into a class with 
            divergent inheritance.
        
        .. warning::
        
            Recasting an object renders the previous Python object inoperable 
            and dead. It will cease to receive updates from the ``HGXLink``, 
            and subsequent manipulation of the old object is likely to cause 
            bugs with the new object as well.

        .. code-block:: python

            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> obj = hgx.JsonObj.recast_threadsafe(obj)
            >>> obj
            <JsonObj with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>

    .. method:: _hgx_push()
                hgx_push_threadsafe()
                hgx_push_loopsafe()
    
        Notifies the Hypergolix service (through the ``HGXLink``) of updates to
        the object. Must be called explicitly for any changes to be available 
        outside of the current Python session.

        :raises hypergolix.exceptions.IPCError: if unsuccessful.
        :raises hypergolix.exceptions.LocallyImmutable: if the object is 
            static, or if the current Hypergolix user did not create it.
        :raises hypergolix.exceptions.DeadObject: if the object is unavailable,
            for example, as a result of a :meth:`_hgx_discard()` call.

        .. code-block:: python

            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> # This state is unknown everywhere except in current memory
            >>> obj.hgx_state = b'Foo'
            >>> obj.hgx_state = b'Bar'
            >>> # Hypergolix now has no record of b'Foo' ever existing.
            >>> obj.hgx_push_threadsafe()
            >>> # The new state b'Bar' is now known to Hypergolix.

    .. method:: _hgx_sync()
                hgx_sync_threadsafe()
                hgx_sync_loopsafe()
    
        Manually initiates an update through Hypergolix. So long as you create 
        and retrieve objects through the ``HGXLink``, you will not need these 
        methods.

        :raises hypergolix.exceptions.IPCError: if unsuccessful.
        :raises hypergolix.exceptions.DeadObject: if the object is unavailable,
            for example, as a result of a :meth:`_hgx_discard()` call.

        .. code-block:: python

            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> obj.hgx_sync_threadsafe()

    .. method:: _hgx_share(recipient)
                hgx_share_threadsafe(recipient)
                hgx_share_loopsafe(recipient)
    
        Shares the ``ObjBase`` instance with ``recipient``. The recipient will 
        receive a read-only copy of the object, which will automatically update 
        upon any local changes that are :meth:`_hgx_push()`\ ed upstream.

        :param Ghid recipient: The public key fingerprint "identity" of the 
            entity to share with.
        :raises hypergolix.exceptions.IPCError: if immediately unsuccessful. 
        :raises hypergolix.exceptions.DeadObject: if the object is unavailable,
            for example, as a result of a :meth:`_hgx_discard()` call.
        :raises hypergolix.exceptions.Unsharable: if the object is 
            :attr:`hgx_private`\ .
            
        .. note::
            
            Successful sharing does **not** imply successful receipt.
            The recipient could ignore the share, be temporarily unavailable, 
            etc.
            
        .. note::
        
            In order to actually receive the object, the recipient must have a 
            share handler defined for the :attr:`hgx_api_id` of the object.

        .. code-block:: python

            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> bob = hgx.Ghid.from_str('AfhB1mAR7U4Uq-UiFg9zCgIIocqmxiSnRPe5orzAjPPh71ChXWRFhwl3scgAM6w-iVXdzLVYHc-MDg4Dfx5dSVE=')
            >>> obj.hgx_share_threadsafe(bob)

    .. method:: _hgx_freeze()
                hgx_freeze_threadsafe()
                hgx_freeze_loopsafe()
    
        Creates a static "snapshot" of a dynamic object. This new static object 
        will be available at its own dedicated address.

        :returns: a frozen copy of the ``ObjBase`` (or subclass) instance. The
            class of the new instance will match the class of the original.
        :raises hypergolix.exceptions.IPCError: if unsuccessful.
        :raises hypergolix.exceptions.LocallyImmutable: if the object is 
            static.
        :raises hypergolix.exceptions.DeadObject: if the object is unavailable,
            for example, as a result of a :meth:`_hgx_discard()` call.

        .. code-block:: python

            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> obj.hgx_dynamic
            True
            >>> frozen = obj.hgx_freeze_threadsafe()
            >>> frozen
            <ObjBase with state b'hello world' at Ghid('ARS48N5rz9V0np816B_vZaRNSLd5PBQUXawu6NCYyMiZbSowffC3IZUJBGOAhX3WS1IyTMmaGOUhonNSJgzI8VE=')>
            >>> frozen.hgx_dynamic
            False

    .. method:: _hgx_hold()
                hgx_hold_threadsafe()
                hgx_hold_loopsafe()
    
        Creates a separate binding to the object, preventing its deletion. This 
        does not necessarily prevent other applications at the 
        currently-logged-in Hypergolix user session from removing the object.

        :raises hypergolix.exceptions.IPCError: if unsuccessful.
        :raises hypergolix.exceptions.DeadObject: if the object is unavailable,
            for example, as a result of a :meth:`_hgx_discard()` call.

        .. code-block:: python

            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> obj.hgx_hold_threadsafe()

    .. method:: _hgx_discard()
                hgx_discard_threadsafe()
                hgx_discard_loopsafe()
    
        Notifies the Hypergolix service that the application is no longer 
        interested in the object, but does not delete it. This renders the 
        object inoperable and dead, preventing most future operations. However, 
        a new copy of the object can still be retrieved through any of the 
        :meth:`HGXLink._get()` methods.

        :raises hypergolix.exceptions.IPCError: if unsuccessful.
        :raises hypergolix.exceptions.DeadObject: if the object is unavailable,
            for example, as a result of a :meth:`_hgx_discard()` call.

        .. code-block:: python

            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> obj.hgx_discard_threadsafe()

    .. method:: _hgx_delete()
                hgx_delete_threadsafe()
                hgx_delete_loopsafe()
    
        Attempts to permanently delete the object. If successful, it will be 
        inoperable and dead. It will also be removed from Hypergolix and made 
        unavailable to other applications, as well as unavailable to any 
        recipients of an :meth:`_hgx_share()` call, unless they have called 
        :meth:`_hgx_hold()`.

        :raises hypergolix.exceptions.IPCError: if unsuccessful.
        :raises hypergolix.exceptions.DeadObject: if the object is unavailable,
            for example, as a result of a :meth:`_hgx_discard()` call.

        .. code-block:: python

            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> obj.hgx_delete_threadsafe()

-------------------------------------------------------------------------------
Hypergolix  proxies
-------------------------------------------------------------------------------

.. class:: ProxyBase(hgxlink, state, api_id, dynamic, private, ghid=None, binder=None)

    .. versionadded:: 0.1
    
    The Hypergolix proxy, partly inspired by ``weakref.proxy``, is a mechanism
    by which almost any existing Python object can be encapsulated within a 
    Hypergolix-aware wrapper. In every other way, the proxy behaves exactly 
    like the original object. This is accomplished by overloading the 
    ``ProxyBase.__getattr__()``, ``ProxyBase.__setattr__()``, and 
    ``ProxyBase.__delattr__()`` methods.
    
    .. note::
    
        :class:`ProxyBase` is a subclass of :class:`ObjBase`. Unless otherwise 
        stated, it exposes the same API as :class:`ObjBase`, **in addition** to 
        the API of the proxied object.

    A proxy is hashable if its :attr:`ObjBase.hgx_ghid` is defined, but 
    unhashable otherwise. Note, however, that this hash has nothing to do with
    the proxied object. Also note that 
    ``isinstance(proxy_obj, collections.Hashable)`` will always identify an 
    :class:`ObjProxy` as hashable, regardless of its actual runtime behavior.

    :param HGXLink hgxlink: The currently-active :class:`HGXLink` object used 
        to connect to the Hypergolix service.
    :param state: The state of the object. For objects using the default (*ie* 
        noop) serialization, this must be ``bytes``-like. For subclasses of 
        ``ObjBase``, this can be anything supported by the subclass' 
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
    :returns: The ``ObjBase`` instance, with state declared, **but not 
        initialized with Hypergolix.**

    .. warning::

        As with :class:`ObjBase` objects, :class:`ProxyBase` objects are not 
        intended to be created directly.
        
    .. note::
        
        Support for Python special methods (aka "magic methods", "dunder 
        methods", etc) *is* provided. However, due to implementation details in 
        Python itself, this is accomplished by explicitly passing **all** 
        possible ``__dunder__`` methods *used by Python* to the proxied object.
        
        This has the result that IDEs will present a *very* long list of 
        available methods for :class:`ProxyBase` objects, even if these methods 
        are not, in fact, available. **However, the built-in** ``dir()`` **command 
        should still return a list limited to the methods actually supported by 
        the proxied:proxy combination.**
        
    .. note::
    
        Proxy objects will detect other :class:`ProxyBase` instances and 
        subclasses, but **they will not detect** :class:`ObjBase` instances or 
        subclasses unless they also subclass :class:`ProxyBase`. This is 
        intentional behavior.

    .. code-block:: python
 
        >>> obj = hgxlink.new_threadsafe(
        ...     cls = hgx.ProxyBase,
        ...     state = b'Hello world!'
        ... )
        >>> obj
        <ProxyBase to b'hello world' at Ghid('AbJQMjM7sm9YomsiVmKz4b0hAgBuW-YLEzkh-0tgQcEKtrsq4zth-1OTXVWyxaSg4B7-D2SQS5_gycEEwlWtrJc=')>
        >>> obj += b' foo'
        >>> obj 
        <ProxyBase to b'hello world foo' at Ghid('AbJQMjM7sm9YomsiVmKz4b0hAgBuW-YLEzkh-0tgQcEKtrsq4zth-1OTXVWyxaSg4B7-D2SQS5_gycEEwlWtrJc=')>
        >>> obj.hgx_state = b'bar'
        >>> obj
        <ProxyBase to b'bar' at Ghid('AbJQMjM7sm9YomsiVmKz4b0hAgBuW-YLEzkh-0tgQcEKtrsq4zth-1OTXVWyxaSg4B7-D2SQS5_gycEEwlWtrJc=')>
        >>> # This will show all available EVERYTHING for both proxy and proxied
        >>> dir(obj)
        ['_ALL_METAD_3141592', '_HASHMIX_3141592', '__add__', '__class__', '__contains__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattr__', '__getattribute__', '__getitem__', '__getnewargs__', '__gt__', '__hash__', '__init__', '__iter__', '__le__', '__len__', '__lt__', '__mod__', '__module__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__rmod__', '__rmul__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', '_api_id_3141592', '_binder_3141592', '_callback_3141592', '_dynamic_3141592', '_force_delete_3141592', '_force_pull_3141592', '_ghid_3141592', '_hgx_DEFAULT_API_ID', '_hgx_delete', '_hgx_discard', '_hgx_freeze', '_hgx_hold', '_hgx_pack', '_hgx_push', '_hgx_recast', '_hgx_register_callback', '_hgx_share', '_hgx_sync', '_hgx_unpack', '_hgxlink_3141592', '_isalive_3141592', '_private_3141592', '_proxy_3141592', '_render_inop_3141592', '_renormalize_api_id_3141592', 'capitalize', 'center', 'count', 'decode', 'endswith', 'expandtabs', 'find', 'fromhex', 'hex', 'hgx_api_id', 'hgx_binder', 'hgx_clear_callback', 'hgx_delete_loopsafe', 'hgx_delete_threadsafe', 'hgx_discard_loopsafe', 'hgx_discard_threadsafe', 'hgx_dynamic', 'hgx_freeze_loopsafe', 'hgx_freeze_threadsafe', 'hgx_ghid', 'hgx_hold_loopsafe', 'hgx_hold_threadsafe', 'hgx_isalive', 'hgx_persistence', 'hgx_private', 'hgx_push_loopsafe', 'hgx_push_threadsafe', 'hgx_recast_loopsafe', 'hgx_recast_threadsafe', 'hgx_register_callback_loopsafe', 'hgx_register_callback_threadsafe', 'hgx_share_loopsafe', 'hgx_share_threadsafe', 'hgx_state', 'hgx_sync_loopsafe', 'hgx_sync_threadsafe', 'index', 'isalnum', 'isalpha', 'isdigit', 'islower', 'isspace', 'istitle', 'isupper', 'join', 'ljust', 'lower', 'lstrip', 'maketrans', 'partition', 'replace', 'rfind', 'rindex', 'rjust', 'rpartition', 'rsplit', 'rstrip', 'split', 'splitlines', 'startswith', 'strip', 'swapcase', 'title', 'translate', 'upper', 'zfill']
        >>> # But a little manipulation shows us only the proxied obj contents
        >>> set(dir(obj)) - set(dir(hgx.ProxyBase))
        {'__getnewargs__', 'capitalize', 'center', 'count', 'decode', 'endswith', 'expandtabs', 'find', 'fromhex', 'hex', 'index', 'isalnum', 'isalpha', 'isdigit', 'islower', 'isspace', 'istitle', 'isupper', 'join', 'ljust', 'lower', 'lstrip', 'maketrans', 'partition', 'replace', 'rfind', 'rindex', 'rjust', 'rpartition', 'rsplit', 'rstrip', 'split', 'splitlines', 'startswith', 'strip', 'swapcase', 'title', 'translate', 'upper', 'zfill'}

    .. attribute:: hgx_state

        The read-write value of the object itself. This will be serialized and 
        uploaded through Hypergolix upon any call to :meth:`hgx_push()`.
        
        .. note::
        
            Because of how Python works, explicitly reassigning 
            :attr:`hgx_state` is the only way to reassign the value of the 
            proxied object directly. For example, this will fail, overwriting 
            the name of the object, and leaving the original unchanged::
            
                >>> obj
                <ProxyBase to b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
                >>> obj = b'Hello Hypergolix!'
                >>> obj
                b'Hello Hypergolix!'
                
            whereas this will succeed in updating the object state::
            
                >>> obj
                <ProxyBase to b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
                >>> obj.hgx_state = b'Hello Hypergolix!'
                >>> obj
                <ProxyBase to b'Hello Hypergolix!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
        
        .. warning::
            
            Updating ``hgx_state`` will **not** update Hypergolix. To upload 
            the change, you must explicitly call :meth:`ObjBase._hgx_push()`.
        
        :rtype: bytes

        .. code-block:: python

            >>> obj
            <ProxyBase to b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> obj.hgx_state
            b'Hello world!'
            >>> # This change won't yet exist anywhere else
            >>> obj.hgx_state = b'Hello Hypergolix!'
            >>> obj
            <ProxyBase to b'Hello Hypergolix!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>

    .. method:: __eq__(other)
    
        Compares the ``ProxyBase`` with another object. The comparison 
        recognizes other :class:`ObjBase` instances and subclasses (including
        :class:`ProxyBase`), comparing them more thoroughly than other objects.
        
        If the ``other`` object is an :class:`ObjBase` instance, or an instance 
        of one of its subclasses, the comparison will return ``True`` if and 
        only if:
        
        1.  The :attr:`ObjBase.hgx_ghid` attribute compares equally
        2.  The :attr:`ObjBase.hgx_state` attribute compares equally
        3.  The :attr:`ObjBase.hgx_binder` attribute compares equally
        
        If, on the other hand, the ``other`` object is not a Hypergolix object 
        or proxy, it will directly compare ``other`` with :attr:`hgx_state`.

        :param other: The ``ObjBase`` (or subclass) instance to compare 
            with.
        :rtype: bool

        .. code-block:: python

            >>> obj
            <ProxyBase to b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            >>> obj2
            <ProxyBase to b'Hello world!' at Ghid('AWFUmWQJvo3U81-hH3WgtXa9bhB9dyXf1QT0yB_l3b6XwjB-WqeN-Lz7JzkMckhDRcjCFS1EmxrcQ1OE2f0Jxh4=')>
            >>> obj == obj2
            False
            >>> not_hgx_obj = b'Hello world!'
            >>> not_hgx_obj == obj
            True
            >>> obj2 == not_hgx_obj
            True
