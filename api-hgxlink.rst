hypergolix.Ghid and hypergolix.HGXLink
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

.. class:: ConcatKDFHMAC(algorithm, length, salt, otherinfo, backend)

    .. versionadded:: 0.1

    Similar to ConcatKFDHash but uses an HMAC function instead.

    .. warning::

        ConcatKDFHMAC should not be used for password storage.

    .. code-block:: python

        >>> import os
        >>> from cryptography.hazmat.primitives import hashes
        >>> from cryptography.hazmat.primitives.kdf.concatkdf import ConcatKDFHMAC
        >>> from cryptography.hazmat.backends import default_backend
        >>> backend = default_backend()
        >>> salt = os.urandom(16)
        >>> otherinfo = b"concatkdf-example"
        >>> ckdf = ConcatKDFHMAC(
        ...     algorithm=hashes.SHA256(),
        ...     length=256,
        ...     salt=salt,
        ...     otherinfo=otherinfo,
        ...     backend=backend
        ... )
        >>> key = ckdf.derive(b"input key")
        >>> ckdf = ConcatKDFHMAC(
        ...     algorithm=hashes.SHA256(),
        ...     length=256,
        ...     salt=salt,
        ...     otherinfo=otherinfo,
        ...     backend=backend
        ... )
        >>> ckdf.verify(b"input key", key)

    :param algorithm: An instance of
        :class:`~cryptography.hazmat.primitives.hashes.HashAlgorithm`.

    :param int length: The desired length of the derived key in bytes. Maximum
        is ``hashlen * (2^32 -1)``.

    :param bytes salt: A salt. Randomizes the KDF's output. Optional, but
        highly recommended. Ideally as many bits of entropy as the security
        level of the hash: often that means cryptographically random and as
        long as the hash output. Does not have to be secret, but may cause
        stronger security guarantees if secret; If ``None`` is explicitly
        passed a default salt of ``algorithm.block_size`` null bytes will be
        used.

    :param bytes otherinfo: Application specific context information.
        If ``None`` is explicitly passed an empty byte string will be used.

    :param backend: An instance of
        :class:`~cryptography.hazmat.backends.interfaces.HMACBackend`.

    :raises cryptography.exceptions.UnsupportedAlgorithm: This is raised if the
        provided ``backend`` does not implement
        :class:`~cryptography.hazmat.backends.interfaces.HMACBackend`

    :raises TypeError: This exception is raised if ``salt`` or ``otherinfo``
        is not ``bytes``.

    .. method:: derive(key_material)

        :param bytes key_material: The input key material.
        :return bytes: The derived key.
        :raises TypeError: This exception is raised if ``key_material`` is not
                           ``bytes``.

        Derives a new key from the input key material.

    .. method:: verify(key_material, expected_key)

        :param bytes key_material: The input key material. This is the same as
                                   ``key_material`` in :meth:`derive`.
        :param bytes expected_key: The expected result of deriving a new key,
                                   this is the same as the return value of
                                   :meth:`derive`.
        :raises cryptography.exceptions.InvalidKey: This is raised when the
                                                    derived key does not match
                                                    the expected key.
        :raises cryptography.exceptions.AlreadyFinalized: This is raised when
                                                          :meth:`derive` or
                                                          :meth:`verify` is
                                                          called more than
                                                          once.

        This checks whether deriving a new key from the supplied
        ``key_material`` generates the same key as the ``expected_key``, and
        raises an exception if they do not match.