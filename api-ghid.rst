hypergolix.\ :class:`Ghid`
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

    .. code-block:: python

        >>> from hypergolix import Ghid
        >>> ghid = Ghid(1, bytes(64))
        >>> ghid
        Ghid(algo=1, address=b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00')

    .. attribute:: algo

        The Golix-specific ``int`` identifier for the hash algorithm.

    .. attribute:: address

        The hash digest of the Ghid, in ``bytes``.
        
        :rtype: bytes

    .. method:: __eq__(other)
    
        Compares with another :class:`Ghid` instance.

        :param Ghid other: The ``Ghid`` instance to compare with.
        :rtype: bool
        :raises TypeError: when attempting to compare with a non-Ghid-like 
            object.

    .. method:: __str__()
    
        Returns a string representation of the ``Ghid`` *object*, 
        including its class, using a truncated url-safe base64-encoded version
        of its bytes serialization.

        :rtype: str 

        .. code-block:: python

            >>> ghid
            Ghid(algo=1, address=b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00')
            >>> str(ghid)
            Ghid('AQAAA...')

    .. method:: __bytes__()
    
        Serializes the ``Ghid`` into a Golix-compliant bytestring.

        :rtype: bytes

        .. code-block:: python

            >>> ghid
            Ghid(algo=1, address=b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00')
            >>> bytes(ghid)
            b'\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'
            
    .. classmethod:: from_bytes(data)
    
        Loads a ``Ghid`` from a Golix-compliant bytestring.
    
        :param bytes data: The serialization to load
        :rtype: Ghid

        .. code-block:: python

            >>> ghid
            Ghid(algo=1, address=b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00')
            >>> bytes(ghid)
            b'\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'
            >>> ghid2 = Ghid.from_bytes(b'\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00')
            >>> ghid2 == ghid
            True

    .. method:: as_str()
        
        Returns the raw url-safe base64-encoded version of the
        ``Ghid``'s serialization, without a class identifier.
        
        :rtype: str 

        .. code-block:: python

            >>> ghid
            Ghid(algo=1, address=b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00')
            >>> ghid.as_str()
            'AQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA='
            
    .. classmethod:: from_str(b64)
    
        Loads a ``Ghid`` from a url-safe base64-encoded Golix-compliant 
        bytestring.
    
        :param str b64: The serialization to load
        :rtype: Ghid

        .. code-block:: python

            >>> ghid
            Ghid(algo=1, address=b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00')
            >>> ghid.as_str()
            'AQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA='
            >>> ghid3 = Ghid.from_str('AQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=')
            >>> ghid3 == ghid
            True
            
    .. classmethod:: pseudorandom(algo)
    
        Creates a pseudorandom ``Ghid`` for the passed ``int`` algorithm
        identifier.
    
        :param str b64: The serialization to load
        :rtype: Ghid
        
        .. warning::
        
            This is not suitable for cryptographic purposes. It is primarily
            useful during testing.

        .. code-block:: python

            >>> ghid
            Ghid(algo=1, address=b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00')
            >>> ghid.as_str()
            'AQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA='
            >>> ghid3 = Ghid.from_str('AQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=')
            >>> ghid3 == ghid
            True