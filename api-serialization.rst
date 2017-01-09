===============================================================================
Serialized Python objects
===============================================================================

.. note::

    This assumes familiarity with :class:`HGXLink`, :class:`Ghid`, 
    :class:`Obj`, and :class:`Proxy` objects.

-------------------------------------------------------------------------------
JSON serialization
-------------------------------------------------------------------------------

.. class:: JsonObj(state, api_id, dynamic, private, *, hgxlink, ipc_manager, _legroom, ghid=None, binder=None, callback=None)

    .. versionadded:: 0.1
    
    A Hypergolix object that uses the built-in ``json`` library for 
    serialization. The resulting string is then encoded in UTF-8. Use it 
    exactly as you would any other :class:`Obj` object.
        
    .. warning::
    
        ``TypeError``\ s as a result of improper ``state`` declarations 
        will not be reported until their value is pushed upstream via 
        :meth:`Obj.push()` or equivalent.

    .. code-block:: python

        >>> obj = hgxlink.new_threadsafe(
        ...     cls = hgx.JsonObj,
        ...     state = 'Hello Json!',
        ... )
        >>> obj
        <JsonObj with state 'Hello Json!' at Ghid('bf3dR')>
        >>> obj.state = 5
        >>> obj
        <JsonObj with state 5 at Ghid('bf3dR')>
        >>> obj.state = {'seven': 7}
        >>> obj
        <JsonObj with state {'seven': 7} at Ghid('bf3dR')>

.. class:: JsonProxy(state, api_id, dynamic, private, *, hgxlink, ipc_manager, _legroom, ghid=None, binder=None, callback=None)

    .. versionadded:: 0.1
    
    A Hypergolix proxy that uses the built-in ``json`` library for 
    serialization. The resulting string is then encoded in UTF-8. Use it 
    exactly as you would any other :class:`Proxy` object.
        
    .. warning::
    
        ``TypeError``\ s as a result of improper ``hgx_state`` declarations 
        will not be reported until their value is pushed upstream via 
        :meth:`Obj._hgx_push()` or equivalent.

    .. code-block:: python

        >>> obj = hgxlink.new_threadsafe(
        ...     cls = hgx.JsonProxy,
        ...     state = 'Hello Json!',
        ... )
        >>> obj
        <JsonProxy to 'Hello Json!' at Ghid('bf3dR')>
        >>> obj.hgx_state = 5
        >>> obj
        <JsonProxy to 5 at Ghid('bf3dR')>
        >>> obj.hgx_state = {'seven': 7}
        >>> obj
        <JsonProxy to {'seven': 7} at Ghid('bf3dR')>

-------------------------------------------------------------------------------
Pickle serialization
-------------------------------------------------------------------------------

.. class:: PickleObj(state, api_id, dynamic, private, *, hgxlink, ipc_manager, _legroom, ghid=None, binder=None, callback=None)

    .. versionadded:: 0.1
    
    A Hypergolix object that uses the built-in ``pickle`` library for 
    serialization. The resulting string is then encoded in UTF-8. Use it 
    exactly as you would any other :class:`Obj` object.

    .. danger::

        Never use ``pickle`` to de/serialize objects from an untrusted source. 
        Because ``pickle`` allows objects to control their own deserialization, 
        retrieving such an object effectively gives the object creator full 
        control over your computer (within the privilege limits of the current 
        Python process).
        
    .. warning::
    
        ``TypeError``\ s as a result of improper ``state`` declarations 
        will not be reported until their value is pushed upstream via 
        :meth:`Obj.push()` or equivalent.

    .. code-block:: python

        >>> obj = hgxlink.new_threadsafe(
        ...     cls = hgx.PickleObj,
        ...     state = 'Hello Pickle!',
        ... )
        >>> obj
        <PickleObj with state 'Hello Pickle!' at Ghid('bf3dR')>
        >>> obj.state = 5
        >>> obj
        <PickleObj with state 5 at Ghid('bf3dR')>
        >>> obj.state = {'seven': 7}
        >>> obj
        <PickleObj with state {'seven': 7} at Ghid('bf3dR')>

.. class:: PickleProxy(state, api_id, dynamic, private, *, hgxlink, ipc_manager, _legroom, ghid=None, binder=None, callback=None)

    .. versionadded:: 0.1
    
    A Hypergolix proxy that uses the built-in ``pickle`` library for 
    serialization. The resulting string is then encoded in UTF-8. Use it 
    exactly as you would any other :class:`Proxy` object.

    .. danger::

        Never use ``pickle`` to de/serialize objects from an untrusted source. 
        Because ``pickle`` allows objects to control their own deserialization, 
        retrieving such an object effectively gives the object creator full 
        control over your computer (within the privilege limits of the current 
        Python process).
        
    .. warning::
    
        ``TypeError``\ s as a result of improper ``hgx_state`` declarations 
        will not be reported until their value is pushed upstream via 
        :meth:`Obj.hgx_push()` or equivalent.

    .. code-block:: python

        >>> obj = hgxlink.new_threadsafe(
        ...     cls = hgx.PickleProxy,
        ...     state = 'Hello Pickle!',
        ... )
        >>> obj
        <PickleProxy to 'Hello Pickle!' at Ghid('bf3dR')>
        >>> obj.hgx_state = 5
        >>> obj
        <PickleProxy to 5 at Ghid('bf3dR')>
        >>> obj.hgx_state = {'seven': 7}
        >>> obj
        <PickleProxy to {'seven': 7} at Ghid('bf3dR')>

-------------------------------------------------------------------------------
Custom serialization
-------------------------------------------------------------------------------

Custom serialization of objects can be easily added to Hypergolix by 
subclassing :class:`Obj` or :class:`Proxy` and overriding:

1.  class attribute ``_hgx_DEFAULT_API``
2.  ``staticmethod`` or ``classmethod`` **coroutine** ``hgx_pack(state)``
3.  ``staticmethod`` or ``classmethod`` **coroutine** ``hgx_unpack(packed)``

A (non-functional) toy example follows:

.. code-block:: python

    from hgx.utils import ApiID
    from hgx import Obj
    from hgx import Proxy

    class ToySerializer:
        ''' An Obj that customizes serialization.
        '''
        _hgx_DEFAULT_API = ApiID(bytes(63) + b'\x04')
        
        @staticmethod
        async def hgx_pack(state):
            ''' Packs the state into bytes.
            '''
            return bytes(state)
        
        @staticmethod
        async def hgx_unpack(packed):
            ''' Unpacks the state from bytes.
            '''
            return object(packed)
            
    
    class ToyObj(ToySerializer, Obj):
        pass
    
    
    class ToyProxy(ToySerializer, Proxy):
        pass
