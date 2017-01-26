===============================================================================
Running Hypergolix
===============================================================================

Before running Hypergolix, make sure you have installed it, as described in
:doc:`setup-1-installing`.

-------------------------------------------------------------------------------
Configuring Hypergolix
-------------------------------------------------------------------------------

The Hypergolix daemon uses a simple YAML file to store its persistent
configuration. This configuration is preferably stored in a subdirectory of
your user folder, ie:

+   ``C:\Users\<username>\.hypergolix\`` for Windows systems
+   ``~/.hypergolix/`` for Unix systems

However, Hypergolix can look for the configuration file in other locations as
well. Specifically, it searches these locations for ``hypergolix.yml``:

1.  at the path specified in the environment variable ``HYPERGOLIX_HOME``
2.  the current directory
3.  ``~/.hypergolix`` (Unix) or ``%HOMEPATH%\.hypergolix`` (Windows)
4.  ``/etc/hypergolix`` (Unix) or ``%LOCALAPPDATA%\Hypergolix`` (Windows)

All configuration of Hypergolix is done through this file. When you first run
Hypergolix, it will automatically create the following default configuration:

+   No remotes (local-only storage)
+   Info-level logging
+   IPC port 7772

.. note::

    You must restart Hypergolix for any configuration changes to take effect.

.. warning::
    
    The config file also stores the ``user id`` and ``fingerprint`` for the
    current Hypergolix user. Unless you record your user id somewhere else (for
    example, in a password manager), if you lose this file, you will probably
    lose access to your Hypergolix account.
    
    You can also see these values by running the command
    ``hypergolix config --whoami``.

Miscellaneous commands:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    hypergolix config [-h]
                      [--whoami]
                      [--register]

-h, --help      Shows the help message and exits
--whoami        Print the fingerprint and user ID for the current
                Hypergolix configuration.
--register      Register the current Hypergolix user, allowing them
                access to the hgx.hypergolix.com remote persister.
                Requires a web browser.

An example ``hypergolix.yml`` file:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: yaml

    process:
      ghidcache: C:\Users\WinUser\.hypergolix\ghidcache
      logdir: C:\Users\WinUser\.hypergolix\logs
      pid_file: C:\Users\WinUser\.hypergolix\hypergolix.pid
      ipc_port: 7772
    instrumentation:
      verbosity: debug
      debug: true
      traceur: false
    user:
      fingerprint: AbYly3OIxHORt5knuFOc7rHSj8-x4cihF3Lbf6tabx6WFRUAqV0gGe89dO0pk9ZNOEeDs5XYshcGfv3Z__vxkco=
      user_id: AcTsUOCtK-7iuyLtTnlwL6fgZ7iiw5v2hHmGpHWnoH5pJzECihIDtXn_AoqgrCnYTu0mx4RdAq9ymbzkFPy5zBQ=
      root_secret: null
    remotes:
    - host: hgx.hypergolix.com
      port: 443
      tls: true
    - host: 123.123.123.123
      port: 7770
      tls: false
    server:
      ghidcache: C:\Users\WinUser\.hypergolix\ghidcache
      logdir: C:\Users\WinUser\.hypergolix\logs
      pid_file: C:\Users\WinUser\.hypergolix\hgx-server.pid
      host: AUTO
      port: 7770
      verbosity: debug
      debug: true

Process configuration:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Hypergolix process can be customized with specific disk locations. You may
also change the localhost port used for inter-process communication with app
integrations.

The ``ghidcache`` directory is used to store the individual Hypergolix objects.
It may be the same as the server ``ghidcache``.

.. warning::

    Though the Hypergolix app and server may share a ``ghidcache`` directory,
    running them from the same directory **at the same time** is currently
    unsupported, and will thoroughly break Hypergolix.
    
The ``logdir`` directory stores a rotating collection of Hypergolix logs. A new
log sequence is created every time Hypergolix starts. It may be necessary to
periodically empty this directory.

The ``pid_file`` is used to store the Hypergolix process ID, and to prevent
multiple instances of the same Hypergolix process from starting.

The ``ipc_port`` setting controls which localhost port is used by Hypergolix
IPC. It defaults to ``7772``.
                        
.. warning::

    Changing the IPC port from the default will require you to always supply
    the correct ``ipc_port`` to the :class:`HGXLink`.
    
.. code-block:: yaml

    process:
      ghidcache: C:\Users\WinUser\.hypergolix\ghidcache
      logdir: C:\Users\WinUser\.hypergolix\logs
      pid_file: C:\Users\WinUser\.hypergolix\hypergolix.pid
      ipc_port: 7772

Instrumentation configuration:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hypergolix has various instrumentation capabilities to aid in diagnosing
problems. All logs are stored locally, in the directory specified in ``logdir``
above.

Verbosity can be configured between the following values, from quietest to
loudest:

1.  ``error`` logs only errors
2.  ``warning`` logs errors and warnings
3.  ``info`` logs errors, warnings, and informational messages
4.  ``debug`` logs all of the above, plus ``hypergolix`` debug messages
5.  ``shouty`` logs all of the above, plus ``websockets`` debug messages
6.  ``extreme`` logs all of the above, plus ``asyncio`` debug messages

Hypergolix can be run in ``debug`` mode, **which will degrade local performance
slightly,** but without it, logged exception tracebacks will be incomplete.

The ``traceur`` key is currently unused.

.. code-block:: yaml

    instrumentation:
      verbosity: debug
      debug: true
      traceur: false

User configuration:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``user`` configuration block sets up the Hypergolix user.

.. danger::

    Tampering with this block can render your Hypergolix account unusable!
    
The ``fingerprint`` field is your :class:`Ghid` fingerprint. Other Hypergolix
accounts can use it to share things with you.

The ``user_id`` field is a :class:`Ghid` reference to the object containing
your account information, including your private keys. Without it, you cannot
access your account.

The ``root_secret`` field can be used for password-less authentication. We
**strongly recommend against using this field** until the login mechanism has
been hardened, and even then, it should only be used for semi- or
fully-autonomous systems that must survive a system reboot without remote
interaction.

.. code-block:: yaml

    user:
      fingerprint: AbYly3OIxHORt5knuFOc7rHSj8-x4cihF3Lbf6tabx6WFRUAqV0gGe89dO0pk9ZNOEeDs5XYshcGfv3Z__vxkco=
      user_id: AcTsUOCtK-7iuyLtTnlwL6fgZ7iiw5v2hHmGpHWnoH5pJzECihIDtXn_AoqgrCnYTu0mx4RdAq9ymbzkFPy5zBQ=
      root_secret: null

Remote configuration:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Remote persistence servers store Hypergolix data nonlocally. For two Hypergolix
accounts to be able to communicate, they must always have at least one
persistence server in common.

You can use any combination of remotes you'd like. To use only local storage
(*ie*, to use no remotes), set the key to an empty list:

.. code-block:: yaml

    remotes: []
    
Otherwise, each remote should be configured as a combination of a host, a port,
and a boolean indicator for whether or not the remote server uses TLS:

.. code-block:: yaml

    remotes:
    - host: hgx.hypergolix.com
      port: 443
      tls: true
      
Server configuration:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The server block allows you to run a remote persistence server on your own
machine. It must be started separately (and in addition to) the Hypergolix app.

The ``ghidcache`` directory is used to store the individual Hypergolix objects.
It may be the same as the app ``ghidcache``.

.. warning::

    Though the Hypergolix app and server may share a ``ghidcache`` directory,
    running them from the same directory **at the same time** is currently
    unsupported, and will thoroughly break Hypergolix.
    
The ``logdir`` directory stores a rotating collection of Hypergolix logs. A new
log sequence is created every time Hypergolix starts. It may be necessary to
periodically empty this directory.

The ``pid_file`` is used to store the Hypergolix process ID, and to prevent
multiple instances of the same Hypergolix process from starting.

The ``host`` field determines which hostname the remote server will bind to. By
default (including when defined as ``null``, it will bind only to
``localhost``. If set to ``AUTO``, Hypergolix will automatically determine the
machine's current IP address, and bind to that. If set to ``ANY``, it will bind
to any hosts at that port.

The ``port`` field determines which port the remote server will bind to. It
defaults to 7770.

1.  ``error`` logs only errors
2.  ``warning`` logs errors and warnings
3.  ``info`` logs errors, warnings, and informational messages
4.  ``debug`` logs all of the above, plus ``hypergolix`` debug messages
5.  ``shouty`` logs all of the above, plus ``websockets`` debug messages
6.  ``extreme`` logs all of the above, plus ``asyncio`` debug messages

Hypergolix can be run in ``debug`` mode, **which will degrade local performance
slightly,** but without it, logged exception tracebacks will be incomplete.

.. code-block:: yaml

    server:
      ghidcache: C:\Users\WinUser\.hypergolix\ghidcache
      logdir: C:\Users\WinUser\.hypergolix\logs
      pid_file: C:\Users\WinUser\.hypergolix\hgx-server.pid
      host: AUTO
      port: 7770
      verbosity: debug
      debug: true

-------------------------------------------------------------------------------
Running Hypergolix
-------------------------------------------------------------------------------

Once installed and configured, Hypergolix is easy to use:

.. code-block:: bash

    # Start the app daemon like this
    hypergolix start app
    
    # Once started, stop the app daemon like this
    hypergolix stop app
    
When you run the Hypergolix app for the first time, it will walk you through
the account creation process. After that, Hypergolix will automatically load
the existing account, prompting you only for your Hypergolix password.

.. warning::

    If you want Hypergolix to connect with other computers, you must configure
    remote(s). See above.
    
.. note::

    Hypergolix is always free to use locally, but on the ``hgx.hypergolix.com``
    remote persistence server, accounts are limited to read-only access (10MB
    up, unlimited down) until they register. Registration currently costs
    $10/month.
    
The Hypergolix server is similarly easy to start. If you want the application 
daemon to be able to connect to your server on startup, you should start the
server first.

.. code-block:: bash

    # Start the server daemon like this
    hypergolix start server
    
    # Once started, stop the server daemon like this
    hypergolix stop server
                
.. note::

    If you are running a Hypergolix server locally, **please enable logging,
    with a verbosity of debug,** and consider enabling debug mode. This will
    help the Hypergolix development team troubleshoot any problems that arise
    during operation.

-------------------------------------------------------------------------------
Using Hypergolix within your application
-------------------------------------------------------------------------------

As mentioned in :doc:`setup-1-installing`, applications should integrate
Hypergolix using the ``hgx`` package on pip:

.. code-block:: bash

    path/to/your/app/env/bin/pip install hgx
    
From here, develop your application as you normally would, importing hgx and
starting the :class:`HGXLink`:

.. code-block:: python

    #!/path/to/your/app/env/bin/python
    import hgx
    hgxlink = hgx.HGXLink()
