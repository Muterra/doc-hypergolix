Hypergolix serialization
===============================================================================

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
