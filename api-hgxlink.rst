===============================================================================
Hypergolix IPC: the :class:`HGXLink`
===============================================================================

.. class:: HGXLink(ipc_port=7772, autostart=True, *args, threaded=True, **kwargs)

    .. versionadded:: 0.1

    The inter-process communications link to the Hypergolix service. Uses 
    Websockets over localhost, by default on port 7772. Runs in a dedicated 
    event loop, typically within a separate thread. Must be explicitly stopped
    during cleanup.

    :param int ipc_port: The localhost port where the Hypergolix service is 
        currently running.
    :param bool autostart: Automatically connect to Hypergolix and start the
        ``HGXLink`` during ``__init__``. If ``False``, the ``HGXLink`` must be
        explicitly started with :meth:`start()`.
    :param \*args: Passed to ``loopa.TaskCommander``.
    :param bool threaded: If ``True``, run the ``HGXLink`` in a separate
        thread; if ``False``, run it in the current thread. In non-threaded
        mode, the ``HGXLink`` will block all operations.
    :param \*\*kwargs: Passed to ``loopa.TaskCommander``.
            
    :returns: The ``HGXLink`` instance.

    .. code-block:: python

        >>> import hgx
        >>> hgxlink = hgx.HGXLink()

    .. attribute:: whoami

        The ``Ghid`` representing the public key fingerprint of the 
        currently-logged-in Hypergolix user. This address may be used for 
        sharing objects. This attribute is read-only.
        
        :return Ghid: if successful
        :raises RuntimeError: if the Hypergolix service is unavailable.

        .. code-block:: python

            >>> hgxlink.whoami
            Ghid(algo=1, address=b'\xf8A\xd6`\x11\xedN\x14\xab\xe5"\x16\x0fs\n\x02\x08\xa1\xca\xa6\xc6$\xa7D\xf7\xb9\xa2\xbc\xc0\x8c\xf3\xe1\xefP\xa1]dE\x87\tw\xb1\xc8\x003\xac>\x89U\xdd\xcc\xb5X\x1d\xcf\x8c\x0e\x0e\x03\x7f\x1e]IQ')

    .. attribute:: token

        The token for the current application (Python session). Only available 
        after registering the application with the Hypergolix service through 
        one of the :meth:`register_token` methods. This attribute is read-only.
        
        :return bytes: if the current application has a token.
        :raises RuntimeError: if the current application has no token.

        .. code-block:: python

            >>> hgxlink.token
            AppToken(b'(\x19i\x07&\xff$!h\xa6\x84\xbcr\xd0\xba\xd1')
        
    .. method:: wrap_threadsafe(callback)
    
        Wraps a blocking/synchronous function for use as a callback. The
        wrapped function will be called from within a single-use,
        dedicated thread from the ``HGXLink``'s internal
        ``ThreadPoolExecutor``, so as not to block the ``HGXLink`` event loop.
        
        This may also be used as a decorator.

        .. code-block:: python
            
            >>> def threadsafe_callback(obj):
            ...     print(obj.state)
            ... 
            >>> threadsafe_callback
            <function threadsafe_callback at 0x00000000051CD620>
            
            >>> # Note that the memory address changes due to wrapping
            >>> hgxlink.wrap_threadsafe(threadsafe_callback)
            <function threadsafe_callback at 0x00000000051CD6A8>
            
            >>> @hgxlink.wrap_threadsafe
            >>> def threadsafe_callback(obj):
            ...     print(obj.state)
            ... 
            >>> threadsafe_callback
            <function threadsafe_callback at 0x000000000520B488>
        
    .. method:: wrap_loopsafe(callback, *, target_loop)
    
        Wraps an asynchronous coroutine for use as a callback. The callback
        will be run in ``target_loop``, which **must be different** from the
        ``HGXLink`` event loop (there is no need to wrap callbacks running
        natively from within the ``HGXLink`` loop). Use this to have the
        ``HGXLink`` run callbacks from within a different event loop (if your
        application is also using ``asyncio`` and providing its own event
        loop).
        
        This may also be used as a decorator.

        .. code-block:: python
            
            >>> async def loopsafe_callback(obj):
            ...     print(obj.state)
            ... 
            >>> loopsafe_callback
            <function loopsafe_callback at 0x0000000005222488>
            
            >>> # Note that the memory address changes due to wrapping
            >>> hgxlink.wrap_loopsafe(loopsafe_callback, target_loop=byo_loop)
            <function loopsafe_callback at 0x00000000051CD6A8>
            
            >>> @hgxlink.wrap_loopsafe(target_loop=byo_loop)
            >>> async def loopsafe_callback(obj):
            ...     print(obj.state)
            ... 
            >>> loopsafe_callback
            <function loopsafe_callback at 0x000000000521A228>
        
    .. method:: start()
    
        Starts the HGXLink, connecting to Hypergolix and obtaining the
        current ``whoami``. Must be called explicitly if ``autostart`` was
        ``False``; otherwise, is called during ``HGXLink.__init__``.

        .. code-block:: python

            >>> hgxlink.start()
            >>> 
    
    .. note::
        
        The following methods each expose three equivalent APIs: 
        
            1.  an API for the HGXLink event loop, written plainly
                (ex: :meth:`register_token()`).
                
                .. warning::
                    
                    This method **must only** be awaited from within the 
                    internal  ``HGXLink`` event loop, or it may break the 
                    ``HGXLink``, and will likely fail to work.
                    
                **This method is a coroutine.** Example usage::
                    
                    token = await register_token()
                
            2.  a threadsafe external API, denoted by the _threadsafe suffix 
                (ex: :meth:`register_token_threadsafe()`). 
                
                .. warning::
                    
                    This method **must not** be called from within the internal 
                    ``HGXLink`` event loop, or it will deadlock.
                
                **This method is a standard, blocking, synchronous method.** 
                Example usage::
                
                    token = register_token_threadsafe()
                
            3.  a loopsafe external API, denoted by the _loopsafe suffix 
                (ex: :meth:`register_token_loopsafe()`). 
                
                .. warning::
                    
                    This method **must not** be awaited from within the 
                    internal ``HGXLink`` event loop, or it will deadlock.
                    
                **This method is a coroutine** that may be awaited from your 
                own external event loop. Example usage::

                    token = await register_token_loopsafe()

    .. method:: stop()
                stop_threadsafe()
                stop_loopsafe()
                
        Called to stop the ``HGXLink`` and disconnect from Hypergolix. Must be
        called before exiting the main thread, or the Python process will not
        exit, and must be manually halted from an operating system process
        manager.
                    
    .. method:: new(cls, state, api_id=None, dynamic=True, private=False)
                new_threadsafe(cls, state, api_id=None, dynamic=True, private=False)
                new_loopsafe(cls, state, api_id=None, dynamic=True, private=False)
                
        Makes a new Hypergolix object.

        :param type cls: the Hypergolix object class to use for this object.
            See :doc:`api-objproxy`.
        :param state: the state to initialize the object with. It will be 
            immediately pushed upstream to Hypergolix during creation of the
            object.
        :param bytes api_id: the API id to use for this object. If ``None``, 
            defaults to the ``cls._hgx_DEFAULT_API``.
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
            ...     cls = hgx.Obj,
            ...     state = b'Hello world!'
            ... )
            >>> obj
            <Obj with state b'Hello world!' at Ghid('Abf3d...')>
            >>> # Get the full address to retrieve the object later
            >>> obj.ghid.as_str()
            'Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk='
                    
    .. method:: get(cls, ghid)
                get_threadsafe(cls, ghid)
                get_loopsafe(cls, ghid)
                
        Retrieves an existing Hypergolix object.

        :param type cls: the Hypergolix object class to use for this object.
            See :doc:`api-objproxy`.
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
            <Obj with state b'Hello world!' at Ghid('Abf3d...')>

    .. method:: register_token(token=None)
                register_token_threadsafe(token=None)
                register_token_loopsafe(token=None)
    
        Requests a new application token from the Hypergolix service or
        re-registers an existing application with the Hypergolix service. If 
        previous instances of the app token have declared a startup object with 
        the Hypergolix service, returns its address.
        
        Tokens can only be registered once per application. Subsequent attempts
        to register a token will raise ``IPCError``. Newly-registered tokens
        will be available at :attr:`token`.
        
        App tokens are required for some advanced features of Hypergolix. This 
        token should be reused whenever (and wherever) that exact application 
        is restarted. It is unique for every application, and every Hypergolix 
        user.

        :param hypergolix.utils.AppToken token: the application's
            pre-registered Hypergolix token, or ``None`` to create one.
        :raises hypergolix.exceptions.IPCError: if unsuccessful.
        :return None: if no startup object has been declared.
        :return hypergolix.Ghid: if a startup object has been declared. This 
            is the address of the object, and can be used in a subsequent
            :meth:`get` call to retrieve it.

        .. code-block:: python

            >>> hgxlink.register_token_threadsafe()
            >>> hgxlink.token
            AppToken(b'(\x19i\x07&\xff$!h\xa6\x84\xbcr\xd0\xba\xd1')
            
            >>> # Some other time, in some other session
            >>> app_token = AppToken(b'(\x19i\x07&\xff$!h\xa6\x84\xbcr\xd0\xba\xd1')
            >>> hgxlink.register_token_threadsafe(app_token)

    .. method:: register_startup(obj)
                register_startup_threadsafe(obj)
                register_startup_loopsafe(obj)
    
        Registers an object as the startup object. Startup objects are useful
        to bootstrap configuration, settings, etc. They can be any Hypergolix
        object, and will be returned to the application at every subsequent
        call to :meth:`register_token`. Startup objects may only be declared
        after registering an app token.

        :param obj: The object to register. May be any Hypergolix object.
        :raises hypergolix.exceptions.UnknownToken: if no token has been
            registered for the application.

        .. code-block:: python
            
            >>> obj = hgxlink.new_threadsafe(Obj, state=b'hello world')
            >>> hgxlink.register_startup_threadsafe(obj)

    .. method:: deregister_startup()
                deregister_startup_threadsafe()
                deregister_startup_loopsafe()
    
        Registers an object as the startup object. Startup objects are useful
        to bootstrap configuration, settings, etc. They can be any Hypergolix
        object, and will be returned to the application at every subsequent
        call to :meth:`register_token`. Startup objects may only be declared
        after registering an app token.

        :raises hypergolix.exceptions.UnknownToken: if no token has been
            registered for the application.
        :raises Exception: if no object has be registered for startup.

        .. code-block:: python
            
            >>> hgxlink.deregister_startup_threadsafe()

    .. method:: register_share_handler(api_id, handler)
                register_share_handler_threadsafe(api_id, handler)
                register_share_handler_loopsafe(api_id, handler)
    
        Registers a handler for incoming, unsolicited object shares from other 
        Hypergolix users. Without registering a share handler, Hypergolix 
        applications cannot receive shared objects from other users.
        
        The share handler will also be called when other applications from the
        same Hypergolix user create an object with the appropriate ``api_id``.
        
        The share handler callback will be invoked with three arguments: the
        :class:`Ghid` of the incoming object, the fingerprint :class:`Ghid` of
        the share origin, and the :class:`hypergolix.utils.ApiID` of the
        incoming object.

        :param hypergolix.utils.ApiID api_id: determines what objects will be
            sent to the  application. Any objects shared with the current
            Hypergolix user with a matching api_id will be sent to the
            application.
        :param handler: the share handler. Unless the ``handler`` can be used
            safely from within the ``HGXLink`` internal event loop, it **must**
            be wrapped through :meth:`wrap_threadsafe` or :meth:`wrap_loopsafe`
            prior to registering it as a share handler.
        :raises TypeError: If the api_id is not :class:`hypergolix.utils.ApiID`
            or the handler is not a coroutine (wrap it using
            :meth:`wrap_threadsafe` or :meth:`wrap_loopsafe` prior to
            registering it as a share handler).
            
        .. warning::
        
            Any given API ID can have at most a single share handler per app. 
            Subsequent calls to any of the :meth:`register_share_handler()` 
            methods will overwrite the existing share handler without warning.

        .. code-block:: python

            >>> @hgxlink.wrap_threadsafe
            ... def handler(ghid, origin, api_id):
            ...     print('Incoming object: ' + str(ghid))
            ...     print('Sent by: ' + str(origin))
            ...     print('With API ID: ' + str(api_id))
            ... 
            >>> hgxlink.register_share_handler_threadsafe(
            ...     api_id = hypergolix.utils.ApiID.pseudorandom(),
            ...     handler = handler
            ... )
