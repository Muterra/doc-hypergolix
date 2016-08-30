hypergolix.Ghid
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