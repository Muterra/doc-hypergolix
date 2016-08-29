hypergolix.Ghid and hypergolix.HGXLink
===============================================================================

.. class:: Ghid(algo, address)

    .. versionadded:: 0.1
    
    The "Golix hash identifier": a unique content address for all Golix and
    Hypergolix content, as defined in the Golix spec. For identities, this is
    approximately equivalent to their public key fingerprint; for static 
    objects, this is the hash digest of their content; for dynamic objects, 
    this is the hash digest of their dynamic pointers (in Golix terminology, 
    their "bindings").
    
    .. note::
        
        ``Ghid`` instances are hashable and may be used as keys in collections.

    :param int algo: The Golix-specific integer identifier for the hash 
        algorithm. Currently, only ``1`` is supported.
    :param bytes address: The hash digest of the Ghid.
    :raises ValueError: for invalid ``algo`` s.
    :raises ValueError: when the length of ``address`` does not match the 
        expected length for the passed ``algo``.
    
    .. warning::
    
        Once created, changing a ``Ghid``'s ``algo`` and ``address`` attributes 
        will break hashing. Avoid doing so. In the future, these attributes 
        will be read-only.

    .. attribute:: algo

        The Golix-specific ``int`` identifier for the hash algorithm.

    .. attribute:: address

        The hash digest of the Ghid, in ``bytes``.

    .. method:: __eq__(other)

        :param Ghid other: The ``Ghid`` instance to compare with.
        :return bool: The comparison result.
        :raises TypeError: when attempting to compare with a non-Ghid-like 
            object.

    .. method:: __bytes__()

        :return bytes: The Golix-compliant bytes serialization of the ``Ghid``.

    .. method:: __str__()

        :return str: Returns a string representation of the ``Ghid`` *object*, 
            including its class, using a url-safe base64-encoded version of its
            bytes serialization.

    .. method:: as_str()

        :return str: Returns the raw url-safe base64-encoded version of the
            ``Ghid``'s serialization, without a class identifier.
            
    .. classmethod:: from_bytes(data)
    
        :param bytes data: The Golix-compliant serialization of the ``Ghid`` to
            load.
        :return Ghid: The resultant ``Ghid`` instance.
            
    .. classmethod:: from_str(b64)
    
        :param str b64: The url-safe base64-encoded serialization of the 
            ``Ghid`` to load.
        :return Ghid: The resultant ``Ghid`` instance.

    .. code-block:: python

        >>> from hypergolix import Ghid
        >>> ghid = Ghid(1, bytes(64))
        >>> ghid
        Ghid(algo=1, address=b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00')
        >>> bytes(ghid)
        b'\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'
        >>> ghid2 = Ghid.from_bytes(b'\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00')
        >>> ghid2 == ghid
        True
        >>> str(ghid)
        Ghid('AQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=')
        >>> ghid.as_str()
        'AQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA='
        >>> ghid3 = Ghid.from_str('AQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=')
        >>> ghid3 == ghid
        True

.. class:: HGXLink(ipc_port=7772, debug=False, aengel=None)

    .. versionadded:: 0.1

    The inter-process communications link to the Hypergolix service. Uses 
    Websockets over localhost, by default on port 7772. Runs in a dedicated 
    event loop within a separate thread. Automatically closes cleanly when the
    main thread exits.

    :param int ipc_port: The localhost port where the Hypergolix service is 
        currently running.
    :param bool debug: Run the link in debug mode.
    :param hypergolix.utils.Aengel aengel: Watches the main thread for closure,
        initiating clean shutdown of the link. If ``None``, creates a dedicated 
        ``Aengel`` instance for this HGXLink. If passed an existing instance of 
        ``Aengel``, the HGXLink will add itself to the watcher.
        
        .. warning::
            Changing this value from its default is not recommended, and may 
            result in improper shutdown, possibly including orphaned threads.
            
    :returns: The ``HGXLink`` instance, after connecting with the Hypergolix 
        service.

    .. code-block:: python

        >>> import hypergolix as hgx
        >>> hgxlink = hgx.HGXLink()

    .. attribute:: whoami

        The ``Ghid`` representing the public key fingerprint of the 
        currently-logged-in Hypergolix user. This address may be used for 
        sharing objects.
        
        :return Ghid: if successful
        :raises RuntimeError: if the Hypergolix service is unavailable.

        .. code-block:: python

            >>> hgxlink.whoami
            Ghid(algo=1, address=b'\xf8A\xd6`\x11\xedN\x14\xab\xe5"\x16\x0fs\n\x02\x08\xa1\xca\xa6\xc6$\xa7D\xf7\xb9\xa2\xbc\xc0\x8c\xf3\xe1\xefP\xa1]dE\x87\tw\xb1\xc8\x003\xac>\x89U\xdd\xcc\xb5X\x1d\xcf\x8c\x0e\x0e\x03\x7f\x1e]IQ')

    .. attribute:: app_token

        The token for the current application (Python session). Only available 
        after registering the application with the Hypergolix service through 
        one of the ``get_new_token`` or ``set_existing_token`` methods.
        
        :return bytes: if the current application has a token.
        :raises RuntimeError: if the current application has no token.

        .. code-block:: python

            >>> hgxlink.app_token
            b'\xe3\xc69\x0f'
        
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
                    
    .. method:: _new(cls, state, api_id=None, dynamic=True, private=False)
                new_threadsafe(cls, state, api_id=None, dynamic=True, private=False)
                new_loopsafe(cls, state, api_id=None, dynamic=True, private=False)
                
        Makes a new Hypergolix object.

        :param type cls: the ``hypergolix.ObjBase`` class or subclass to use 
            for this object.
        :param state: the state to initialize the object with. It will be 
            immediately pushed upstream to Hypergolix during creation of the
            object.
        :param bytes api_id: the API id to use for this object. If ``None``, 
            defaults to the ``_hgx_DEFAULT_API_ID`` declared for the passed 
            ``cls`` .
        :param bool dynamic: determines whether the created object will be 
            dynamic (and therefore mutable), or static (and wholly immutable).
        :param bool private: determines whether the created object will be 
            restricted to **this specific application,** for this specific 
            Hypergolix user. By default, objects created by any Hypergolix 
            application are available to all other Hypergolix apps for the 
            current Hypergolix user.
        :returns: the created object.
        :raises hypergolix.exceptions.IPCError: upon IPC failure, or improper
            object declaration.
        :raises Exception: for serialization failures. The specific exception 
            type is determined by the serialization process itself.

        .. code-block:: python
     
            >>> obj = hgxlink.new_threadsafe(
            ...     cls = hgx.ObjBase,
            ...     state = b'Hello world!'
            ... )
            >>> obj
            <ObjBase with state b'Hello world!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
            
                    
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

    .. method:: _get_new_token()
                get_new_token_threadsafe()
                get_new_token_loopsafe()
    
        Requests a new application token from the Hypergolix service. App 
        tokens are required for some advanced features of Hypergolix. This 
        token should be reused whenever (and wherever) that exact application 
        is restarted. It is unique for every application, and every Hypergolix 
        user.

        :return bytes: the app token.
        :raises hypergolix.exceptions.IPCError: if unsuccessful.

        .. code-block:: python

            >>> hgxlink.get_new_token_threadsafe()
            b'\xe3\xc69\x0f'

    .. method:: _set_existing_token(app_token)
                set_existing_token_threadsafe(app_token)
                set_existing_token_loopsafe(app_token)
    
        Re-registers an existing application with the Hypergolix service. If 
        previous instances of the app token have declared a startup object with 
        the Hypergolix service, returns it.

        :param bytes app_token: the application's pre-registered Hypergolix 
            token.
        :return None: if no startup object has been declared.
        :return hypergolix.ObjBase: if a startup object has been declared. This 
            object may then be recast into any other Hypergolix object.
        :raises hypergolix.exceptions.IPCError: if unsuccessful.

        .. code-block:: python

            >>> hgxlink.set_existing_token_threadsafe(b'\xe3\xc69\x0f')

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