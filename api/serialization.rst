Hypergolix serialization
=========================

JSON serialization
------------------

Pickle serialization
------------------

.. danger::

    Never use Pickle to de/serialize objects from an untrusted source. Because
    pickled objects can control their own deserialization, retrieving an object 
    using Pickled for deserialization allows the object creator full control 
    over your system (within the privilege limits of the currently Python 
    process).

Custom serialization
----------------
