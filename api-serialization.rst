Non-trivial object serialization
===============================================================================

.. note::

    This assumes familiarity with :class:`HGXLink`, :class:`Ghid`, 
    :class:`ObjBase`, and :class:`ProxyBase` objects.

JSON serialization: :class:`JsonObj` and :class:`JsonProxy`
-------------------------------------------------------------------------------

Pickle serialization: :class:`PickleObj` and :class:`PickleProxy`
-------------------------------------------------------------------------------

.. danger::

    Never use Pickle to de/serialize objects from an untrusted source. Because
    Pickle allows objects to control their own deserialization, retrieving such 
    an object effectively gives the object creator full control over your 
    system (within the privilege limits of the current Python process).

Custom serialization
-------------------------------------------------------------------------------
