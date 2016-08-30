hypergolix.ObjBase and hypergolix.ProxyBase
===============================================================================

.. note::
    
    This assumes familiarity with :class:`Ghid` and :class:`HGXLink` objects.

.. class:: ObjBase(hgxlink, state, api_id, dynamic, private, ghid=None, binder=None)

    .. versionadded:: 0.1

    A base class for Hypergolix-connected objects. Does not perform any 
    serialization of its state -- in other words, ``ObjBase`` instance state 
    (the actual value of the object) must be ``bytes``-like.
    
    When first created **through the ``HGXLink``,** the declared state is 
    passed to Hypergolix through a call to :meth:`hgx_push()` to create the 
    object; however, ``ObjBase.__init__()`` does not do this internally. 
    Thereafter, any changes to local state must be explicitly pushed upstream 
    using :meth:`hgx_push()`.
    
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
    
    Hypergolix objects persist nonlocally until explicitly deleted through 
    :meth:`hgx_delete()`.

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
        :meth:`hgx_push()`.
        
    .. note::
        
        The default API ID for base objects with no serialization is::
        
            0x 0000000000000000 0000000000000000 0000000000000000 0000000000000000
               0000000000000000 0000000000000000 0000000000000000 0000000000000001
               
        or, more concisely::
       
            bytes(63) + b'\x01'

    .. attribute:: hgx_state

        The read-write value of the object itself. This will be serialized and 
        uploaded through Hypergolix upon any call to :meth:`hgx_push()`.
        
        .. warning::
            
            Updating ``hgx_state`` will **not** update Hypergolix. To upload 
            the change, you must explicitly call :meth:`hgx_push()`.
        
        :return bytes: read-only state.

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
            >>> str(obj.hgx_ghid)
            Ghid('AfhB1mAR7U4Uq-UiFg9zCgIIocqmxiSnRPe5orzAjPPh71ChXWRFhwl3scgAM6w-iVXdzLVYHc-MDg4Dfx5dSVE=')
        
    .. note::
        
        The following methods each expose three equivalent APIs: 
        
            1.  an internal API, denoted by a leading underscore 
                (ex: ``_get_new_token()``).
                
                .. warning::
                    
                    This method **must only** be awaited from within the 
                    internal  ``HGXLink`` event loop, or it may break the 
                    ``HGXLink``, and will likely fail to work.
                    
                **This method is a coroutine.** Example usage::
                    
                    token = await _get_new_token()
                
            2.  a threadsafe external API, denoted by the _threadsafe suffix 
                (ex: ``get_new_token_threadsafe()``). 
                
                .. warning::
                    
                    This method **must not** be called from within the internal 
                    ``HGXLink`` event loop, or it will deadlock.
                
                **This method is a standard, blocking, synchronous method.** 
                Example usage::
                
                    token = get_new_token_threadsafe()
                
            3.  a loopsafe external API, denoted by the _loopsafe suffix 
                (ex: ``get_new_token_loopsafe()``). 
                
                .. warning::
                    
                    This method **must not** be awaited from within the 
                    internal ``HGXLink`` event loop, or it will deadlock.
                    
                **This method is a coroutine** that may be awaited from your 
                own external event loop. Example usage::

                    token = await get_new_token_loopsafe()
                    
    .. method:: _get(cls, ghid)
                get_threadsafe(cls, ghid)
                get_loopsafe(cls, ghid)
                
        Retrieves an existing Hypergolix object.

        :param type cls: the ``hypergolix.ObjBase`` class or subclass to use 
            for this object.
        :param Ghid ghid: the ``Ghid`` address of the object to retrieve.
        :returns: the retrieved object.
        :raises hypergolix.exceptions.IPCError: upon IPC failure, or improper
            object declaration.
        :raises Exception: for serialization failures. The specific exception 
            type is determined by the serialization process itself.

        .. code-block:: python
     
            >>> address = hgx.Ghid.from_str('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')
            >>> obj = hgxlink.get_threadsafe(
            ...     cls = hgx.ObjBase,
            ...     ghid = address
            ... )
            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>

    .. method:: _register_share_handler(api_id, cls, handler)
                register_share_handler_threadsafe(api_id, cls, handler)
                register_share_handler_loopsafe(api_id, cls, handler, target_loop)
    
        Registers a handler for incoming, unsolicited object shares from other 
        Hypergolix users. Without registering a share handler, Hypergolix 
        applications cannot receive shared objects from other users.

        :param bytes api_id: determines what objects will be sent to the 
            application. Any objects shared with the current Hypergolix user 
            with a matching api_id will be sent to the application. Must have a 
            length of 64 bytes.
        :param type cls: the ``hypergolix.ObjBase`` class or subclass to use 
            for these objects. This determines what ``type`` of object will be 
            delivered to the ``handler``.
        :param handler: the share handler. For threadsafe callbacks, this must 
            be a callable; for async callbacks, it must be an awaitable. Upon 
            receipt of a share, the handler will be passed the object as a 
            single argument.
        :param target_loop: for loopsafe callbacks, the event loop to run the 
            callback in.

        .. code-block:: python

            >>> def handler(obj):
            ...     print(repr(obj))
            ... 
            >>> hgxlink.register_share_handler_threadsafe(
            ...     api_id = hgx.ObjBase._hgx_DEFAULT_API_ID,
            ...     cls = hgx.ObjBase,
            ...     handler = handler
            ... )
            
        Resulting call:

        .. code-block:: python

            >>> 
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            
        .. note::
            
            The :meth:`_register_share_handler()` callback will be awaited from 
            within the internal ``HGXLink`` event loop.
            
        .. note::
            
            The :meth:`register_share_handler_threadsafe()` callback will be 
            called from a dedicated, single-use, disposable thread.
            
        .. note::
            
            The :meth:`register_share_handler_loopsafe()` callback will be 
            called from within the passed ``target_loop``.