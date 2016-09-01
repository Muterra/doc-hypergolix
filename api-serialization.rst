Non-trivial object serialization
===============================================================================

.. note::

    This assumes familiarity with :class:`HGXLink`, :class:`Ghid`, 
    :class:`ObjBase`, and :class:`ProxyBase` objects.

JSON serialization: :class:`JsonObj` and :class:`JsonProxy`
-------------------------------------------------------------------------------

Applications may use the included :class:`JsonObj` and :class:`JsonProxy` to 
support a wider range of ``hgx_state`` value types.

.. class:: JsonObj(hgxlink, state, api_id, dynamic, private, ghid=None, binder=None)

    .. versionadded:: 0.1
    
    A Hypergolix object that uses the built-in ``json`` library for 
    serialization. The resulting string is then encoded in UTF-8. Use it 
    exactly as you would any other :class:`ObjBase` object.
    
    .. note::
    
        This **is** a subclass of :class:`ObjBase`, but **not** a subclass of 
        :class:`ProxyBase`.
        
    .. warning::
    
        ``TypeError``\ s as a result of improper ``hgx_state`` declarations 
        will not be reported until their value is pushed upstream via 
        :meth:`ObjBase._hgx_push()` or equivalent.

    .. code-block:: python

        >>> obj = hgxlink.new_threadsafe(
        ...     cls = hgx.JsonObj,
        ...     state = 'Hello Json!',
        ... )
        >>> obj
        <JsonObj with state 'Hello Json!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
        >>> obj.hgx_state = 5
        >>> obj
        <JsonObj with state 5 at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
        >>> obj.hgx_state = {'seven': 7}
        >>> obj
        <JsonObj with state {'seven': 7} at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>

.. class:: JsonProxy(hgxlink, state, api_id, dynamic, private, ghid=None, binder=None)

    .. versionadded:: 0.1
    
    A Hypergolix proxy that uses the built-in ``json`` library for 
    serialization. The resulting string is then encoded in UTF-8. Use it 
    exactly as you would any other :class:`ProxyBase` object.
    
    .. note::
    
        This **is** a subclass of :class:`ObjBase`, **and** a subclass of 
        :class:`ProxyBase`.
        
    .. warning::
    
        ``TypeError``\ s as a result of improper ``hgx_state`` declarations 
        will not be reported until their value is pushed upstream via 
        :meth:`ObjBase._hgx_push()` or equivalent.

    .. code-block:: python

        >>> obj = hgxlink.new_threadsafe(
        ...     cls = hgx.JsonProxy,
        ...     state = 'Hello Json!',
        ... )
        >>> obj
        <JsonProxy to 'Hello Json!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
        >>> obj.hgx_state = 5
        >>> obj
        <JsonProxy to 5 at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
        >>> obj.hgx_state = {'seven': 7}
        >>> obj
        <JsonProxy to {'seven': 7} at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>

Pickle serialization: :class:`PickleObj` and :class:`PickleProxy`
-------------------------------------------------------------------------------

Applications may use the included :class:`PickleObj` and :class:`PickleProxy` 
to support a wider range of ``hgx_state`` value types.

.. class:: PickleObj(hgxlink, state, api_id, dynamic, private, ghid=None, binder=None)

    .. versionadded:: 0.1
    
    A Hypergolix object that uses the built-in ``pickle`` library for 
    serialization. The resulting string is then encoded in UTF-8. Use it 
    exactly as you would any other :class:`ObjBase` object.

    .. danger::

        Never use ``pickle`` to de/serialize objects from an untrusted source. 
        Because ``pickle`` allows objects to control their own deserialization, 
        retrieving such an object effectively gives the object creator full 
        control over your computer (within the privilege limits of the current 
        Python process).
    
    .. note::
    
        This **is** a subclass of :class:`ObjBase`, but **not** a subclass of 
        :class:`ProxyBase`.
        
    .. warning::
    
        ``TypeError``\ s as a result of improper ``hgx_state`` declarations 
        will not be reported until their value is pushed upstream via 
        :meth:`ObjBase._hgx_push()` or equivalent.

    .. code-block:: python

        >>> obj = hgxlink.new_threadsafe(
        ...     cls = hgx.PickleObj,
        ...     state = 'Hello Pickle!',
        ... )
        >>> obj
        <PickleObj with state 'Hello Pickle!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
        >>> obj.hgx_state = 5
        >>> obj
        <PickleObj with state 5 at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
        >>> obj.hgx_state = {'seven': 7}
        >>> obj
        <PickleObj with state {'seven': 7} at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>

.. class:: PickleProxy(hgxlink, state, api_id, dynamic, private, ghid=None, binder=None)

    .. versionadded:: 0.1
    
    A Hypergolix proxy that uses the built-in ``pickle`` library for 
    serialization. The resulting string is then encoded in UTF-8. Use it 
    exactly as you would any other :class:`ProxyBase` object.

    .. danger::

        Never use ``pickle`` to de/serialize objects from an untrusted source. 
        Because ``pickle`` allows objects to control their own deserialization, 
        retrieving such an object effectively gives the object creator full 
        control over your computer (within the privilege limits of the current 
        Python process).
    
    .. note::
    
        This **is** a subclass of :class:`ObjBase`, **and** a subclass of 
        :class:`ProxyBase`.
        
    .. warning::
    
        ``TypeError``\ s as a result of improper ``hgx_state`` declarations 
        will not be reported until their value is pushed upstream via 
        :meth:`ObjBase._hgx_push()` or equivalent.

    .. code-block:: python

        >>> obj = hgxlink.new_threadsafe(
        ...     cls = hgx.PickleProxy,
        ...     state = 'Hello Pickle!',
        ... )
        >>> obj
        <PickleProxy to 'Hello Pickle!' at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
        >>> obj.hgx_state = 5
        >>> obj
        <PickleProxy to 5 at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>
        >>> obj.hgx_state = {'seven': 7}
        >>> obj
        <PickleProxy to {'seven': 7} at Ghid('Abf3dRNZAPhrqY93q4Q-wG0QvPnP_anV8XfauVMlFOvAgeC5JVWeXTUftJ6tmYveH0stGaAJ0jN9xKriTT1F6Mk=')>

Custom serialization
-------------------------------------------------------------------------------

Custom serialization of objects can be easily added to Hypergolix by 
subclassing :class:`ObjBase` and overriding:

1.  class attribute ``ObjBase._hgx_DEFAULT_API_ID``
2.  ``staticmethod`` or ``classmethod`` **coroutine** ``ObjBase._hgx_pack()``
3.  ``staticmethod`` or ``classmethod`` **coroutine** ``ObjBase._hgx_unpack()``

A (non-functional) toy example follows::

    class ToyObj(ObjBase):
        ''' An ObjBase that customizes serialization.
        '''
        _hgx_DEFAULT_API_ID = bytes(63) + b'\x04'
        
        @staticmethod
        async def _hgx_pack(state):
            ''' Packs the state into bytes.
            '''
            return bytes(state)
        
        @staticmethod
        async def _hgx_unpack(packed):
            ''' Unpacks the state from bytes.
            '''
            return object(packed)

Creating proxy classes for new serializations is trivial; simply subclass the 
serialization class and :class:`ProxyBase`. For example, the :class:`JsonProxy` 
class definition, in its entirety, is::
        
    class JsonProxy(JsonObj, ProxyBase):
        ''' Make a proxy object that serializes with json.
        '''
        pass
