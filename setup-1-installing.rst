===============================================================================
Hypergolix installation
===============================================================================

Hypergolix is written in pure Python, but not all of its dependencies are. One
dependency in particular (pycryptodome, used for password ``scrypting``) is
known to cause issues with Anaconda. As such, we recommend installing
Hypergolix into its own dedicated virtual environment, using stock Python.

Hypergolix requires ``python>=3.5.1``.

-------------------------------------------------------------------------------
Linux
-------------------------------------------------------------------------------

Additional installation requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hypergolix (via its https://cryptography.io dependency) requires OpenSSL 1.0.2.
On Linux, we test against versions 1.0.2j and 1.1.0c. Most recent mainstream
Linux distros should ship with a sufficient OpenSSL version, in which case this
should be adequate install preparation:

.. code-block:: bash

    sudo apt-get install build-essential libssl-dev libffi-dev python3-dev
    
However, if you get any crypto-related errors, it's likely you need to re-link
OpenSSL for ``cryptography``, as described `here
<https://cryptography.io/en/latest/installation/#using-your-own-openssl-on-linux>`_.

For reference, this is our install script for automated testing, which does
require some version muckery:

.. code-block:: bash

    if [ -n "${OPENSSL}" ]; then
      OPENSSL_DIR="ossl-1/${OPENSSL}"
      if [[ ! -f "$HOME/$OPENSSL_DIR/bin/openssl" ]]; then
          curl -O https://www.openssl.org/source/openssl-$OPENSSL.tar.gz
          tar zxf openssl-$OPENSSL.tar.gz
          cd openssl-$OPENSSL
          ./config shared no-asm no-ssl2 no-ssl3 -fPIC --prefix="$HOME/$OPENSSL_DIR"
          # modify the shlib version to a unique one to make sure the dynamic
          # linker doesn't load the system one. This isn't required for 1.1.0 at the
          # moment since our Travis builders have a diff shlib version, but it doesn't hurt
          sed -i "s/^SHLIB_MAJOR=.*/SHLIB_MAJOR=100/" Makefile
          sed -i "s/^SHLIB_MINOR=.*/SHLIB_MINOR=0.0/" Makefile
          sed -i "s/^SHLIB_VERSION_NUMBER=.*/SHLIB_VERSION_NUMBER=100.0.0/" Makefile
          make depend
          make install
          
          # Add new openssl to path
          export PATH="$HOME/$OPENSSL_DIR/bin:$PATH"
          export CFLAGS="-I$HOME/$OPENSSL_DIR/include"
          # rpath on linux will cause it to use an absolute path so we don't need to do LD_LIBRARY_PATH
          export LDFLAGS="-L$HOME/$OPENSSL_DIR/lib -Wl,-rpath=$HOME/$OPENSSL_DIR/lib"
      fi

      cd $TRAVIS_BUILD_DIR
    fi

Recommended installation procedure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo apt-get install python3-venv
    mkdir ~/.hypergolix
    python3 -m venv ~/.hypergolix/hgx-env
    ~/.hypergolix/hgx-env/bin/python -m pip install --upgrade pip
    ~/.hypergolix/hgx-env/bin/pip install hypergolix

-------------------------------------------------------------------------------
OSX
-------------------------------------------------------------------------------

Additional installation requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``Cryptography`` ships with compiled binary wheels on OSX, so installation
should not require any prerequisites, though it may be necessary to update
Python. Additionally, one dependency (``donna25519``) requires the ability to
compile C extensions.

Recommended installation procedure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    mkdir ~/.hypergolix
    python3 -m venv ~/.hypergolix/hgx-env
    ~/.hypergolix/hgx-env/bin/python -m pip install --upgrade pip
    ~/.hypergolix/hgx-env/bin/pip install hypergolix

-------------------------------------------------------------------------------
Windows
-------------------------------------------------------------------------------

Additional installation requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The only Windows prerequisite is Python itself. Because of the namespace
conflicts mentioned above, we recommend running Hypergolix in a dedicated
virtualenv created through stock Python.

Python is available `at Python.org <https://www.python.org/downloads/>`_; be
sure to download Python 3 (**not** 2.7.xx).

Recommended installation procedure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    mkdir %HOMEPATH%/.hypergolix
    python -m venv %HOMEPATH%/.hypergolix/hgx-env
    %HOMEPATH%/.hypergolix/hgx-env/Scripts/python -m pip install --upgrade pip
    %HOMEPATH%/.hypergolix/hgx-env/Scripts/pip install hypergolix

-------------------------------------------------------------------------------
Building from source
-------------------------------------------------------------------------------

Hypergolix itself is pure Python, so this is easy:

.. code-block:: bash
    
    git clone https://github.com/Muterra/py_hypergolix.git ./hgx-src
    /path/to/dest/env/bin/pip install -e ./hgx-src
