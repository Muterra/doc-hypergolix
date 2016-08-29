PickleObj, PickleProxy; JsonObj, JsonProxy; custom serialization
===============================================================================

.. note::

    This assumes familiarity with ``HGXLink``, ``Ghid``, ``ObjBase``, and 
    ``ProxyBase`` objects.

+ from .objproxy import PickleObj
+ from .objproxy import PickleProxy
+ from .objproxy import JsonObj
+ from .objproxy import JsonProxy

JSON serialization
-------------------------------------------------------------------------------

Pickle serialization
-------------------------------------------------------------------------------

.. danger::

    Never use Pickle to de/serialize objects from an untrusted source. Because
    Pickle allows objects to control their own deserialization, retrieving such 
    an object effectively gives the object creator full control over your 
    system (within the privilege limits of the current Python process).

Custom serialization
-------------------------------------------------------------------------------
