===============================================================================
Hypergolix installation
===============================================================================

Hypergolix has two parts: 

1.  the Hypergolix daemon (``pip install hypergolix``)
2.  the Hypergolix integration (``pip install hgx``)

To avoid namespace conflicts in dependencies, the daemon should be run in its
own dedicated Python environment. One dependency in particular (pycryptodome,
used for password ``scrypting``) is known to cause issues with shared
environments, especially Anaconda. If using Anaconda, be sure to
``pip install hypergolix`` within a new, **bare** environment.

``hgx``, on the other hand, is a pure Python package, **including its
dependencies.** As such, it is much easier to install, and you should almost
always use ``hgx`` in your actual application.

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

This will install Hypergolix into a dedicated Python virtual environment within 
``/usr/local/hypergolix``, and then add the hypergolix command as a symlink
in ``/usr/local/bin``. Afterwards, Hypergolix should be available directly
through the command line by simply typing (for example)
``hypergolix start app``.

.. code-block:: bash

    sudo apt-get install python3-venv
    sudo mkdir /usr/local/hypergolix
    sudo python3 -m venv /usr/local/hypergolix/hgx-env
    sudo /usr/local/hypergolix/hgx-env/bin/python -m pip install --upgrade pip
    sudo /usr/local/hypergolix/hgx-env/bin/pip install hypergolix
    sudo ln -s /usr/local/hypergolix/hgx-env/bin/hypergolix /usr/local/bin/hypergolix
    
.. 
    comment out this, because we might use it later
    echo -e "\n#Append Hypergolix to path\nPATH=\"\$PATH:\$HOME/.hypergolix/hgx-env/bin\"" >> ~/.profile

Recommended integration procedure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    path/to/your/app/env/bin/pip install hgx

.. code-block:: python

    #!/path/to/your/app/env/bin/python
    import hgx

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

This will install Hypergolix into a dedicated Python virtual environment within 
``/usr/local/hypergolix``, and then add the hypergolix command as a symlink
in ``/usr/local/bin``. Afterwards, Hypergolix should be available directly
through the command line by simply typing (for example)
``hypergolix start app``.

.. code-block:: bash

    sudo apt-get install python3-venv
    sudo mkdir /usr/local/hypergolix
    sudo python3 -m venv /usr/local/hypergolix/hgx-env
    sudo /usr/local/hypergolix/hgx-env/bin/python -m pip install --upgrade pip
    sudo /usr/local/hypergolix/hgx-env/bin/pip install hypergolix
    sudo ln -s /usr/local/hypergolix/hgx-env/bin/hypergolix /usr/local/bin/hypergolix

Recommended integration procedure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    path/to/your/app/env/bin/pip install hgx

.. code-block:: python

    #!/path/to/your/app/env/bin/python
    import hgx

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

This will install Hypergolix within your program files. It will then add the
Hypergolix bin folder to the *end* of your PATH (meaning everything else will
take precedence over it). You will need to run these commands from within an
elevated (Administrator) command prompt.

.. code-block:: bash

    mkdir "%PROGRAMFILES%/Hypergolix"
    python -m venv "%PROGRAMFILES%/Hypergolix/hgx-env"
    "%PROGRAMFILES%/Hypergolix/hgx-env/Scripts/python" -m pip install --upgrade pip
    "%PROGRAMFILES%/Hypergolix/hgx-env/Scripts/pip" install hypergolix
    "%PROGRAMFILES%/Hypergolix/hgx-env/Scripts/python" -m hypergolix.winpath ^%PROGRAMFILES^%/Hypergolix/hgx-env/Scripts
    set PATH=%PATH%;%PROGRAMFILES%/Hypergolix/hgx-env/Scripts
    
.. warning::

    Windows command prompts do not register updates to environment variables
    after they've been started (they do not handle WS_SettingChange messages).
    As such, ``set PATH=%PATH%;%PROGRAMFILES%/Hypergolix/hgx-env/Scripts``
    needs to be called in any prompt that was open before Hypergolix
    installation. Prompts opened afterwards will automatically load the correct
    ``%PATH%``.

Recommended integration procedure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    path/to/your/app/env/Scripts/pip install hgx

.. code-block:: python

    #!/path/to/your/app/env/Scripts/python
    import hgx

-------------------------------------------------------------------------------
Building from source
-------------------------------------------------------------------------------

Hypergolix itself is pure Python, so this is easy. Make sure you satisfy the
installation requirements listed above, and then clone the source and install
it:

.. code-block:: bash
    
    git clone https://github.com/Muterra/py_hypergolix.git ./hgx-src
    /path/to/dest/env/bin/pip install -e ./hgx-src
