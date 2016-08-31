hypergolix.\ :class:`ProxyBase`
===============================================================================

.. note::
    
    This assumes familiarity with :class:`Ghid`, :class:`HGXLink`, and 
    :class:`ObjBase` objects.

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