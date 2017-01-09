===============================================================================
Running Hypergolix
===============================================================================

Before running Hypergolix, make sure you have installed it, as described in
:doc:`setup-1-installing`.

-------------------------------------------------------------------------------
Configuring Hypergolix
-------------------------------------------------------------------------------

The Hypergolix daemon uses a simple JSON file to store its persistent
configuration. This configuration is stored in a subdirectory of your user
folder, ie:

+   ``C:\Users\<username>\.hypergolix\`` for Windows systems
+   ``~/.hypergolix/`` for Unix systems

When you first run Hypergolix, it will automatically create the
following default configuration:

+   No remotes (local-only storage)
+   Warning-level logging
+   IPC port 7772

.. note::

    You must restart Hypergolix for any configuration changes to take effect.

.. warning::
    
    The config file also stores the ``user id`` and ``fingerprint`` for the
    current Hypergolix user. Unless you record your user id somewhere else (for
    example, in a password manager), if you lose this file, you will probably
    lose access to your Hypergolix account.

.. code-block:: bash

    hypergolix config [-h]
                      [--only {local,hgx}]
                      [--add {hgx}]
                      [--remove {hgx}]
                      [--addhost HOST PORT TLS]
                      [--removehost HOST PORT]
                      [--debug | --no-debug]
                      [--verbosity {extreme,shouty,louder,loud,normal,quiet}]
                      [--ipc-port PORT]
                      [--whoami]
                      [--register]

Miscellaneous commands:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-h, --help      Shows the help message and exits
--whoami        Print the fingerprint and user ID for the current
                Hypergolix configuration.
--register      Register the current Hypergolix user, allowing them
                access to the hgx.hypergolix.com remote persister.
                Requires a web browser.

Configuring remotes:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Remote persistence servers store Hypergolix data nonlocally. For two Hypergolix
accounts to be able to communicate, they must always have at least one
persistence server in common.

-o, --only      Use only a single, named remote (or no remote).
                Valid options: ``local``, ``hgx``
-a, --add       Add a named remote. Cannot be combined with ``--only``.
                Valid options: ``hgx``
-r, --remove    Remove a named remote. Cannot be combined with ``--only``.
                Valid options: ``hgx``

-ah, --addhost
                Add a remote host, of form "hostname port TLS".
                
                Cannot be combined with ``--only``.
                
                Example usage:
                ``hypergolix config --addhost 192.168.0.1 7770 False``

-rh, --removehost
                Remove a remote host, of form "hostname port".
                
                Cannot be combined with ``--only``.
                
                Example usage:
                ``hypergolix config --removehost 192.168.0.1 7770``

Runtime configuration:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hypergolix can be run in debug mode, **which will degrade local performance
slightly,** but without it, logged exception tracebacks will be incomplete.
The logs themselves are stored within the Hypergolix configuration folder. You
can also configure the log verbosity, as well as the TCP port used for
inter-process communication between Hypergolix and any hgx integrations.

--debug         Enables debug mode.
--no-debug      Clears debug mode.

-v, --verbosity
                Specify the logging level. Valid options: ``extreme``, 
                ``shouty``, ``louder``, ``loud``, ``normal``, ``quiet``

-ipc, --ipc-port
                Configure which port to use for Hypergolix IPC.
                        
.. warning::

    Changing the IPC port from the default will require you to always supply
    the correct ``ipc_port`` to the :class:`HGXLink`.

-------------------------------------------------------------------------------
Running the Hypergolix daemon
-------------------------------------------------------------------------------

Once installed and configured, Hypergolix is easy to use:

.. code-block:: bash

    # Start like this
    hypergolix start app
    
    # Once started, stop like this
    hypergolix stop app
    
When you run Hypergolix for the first time, it will walk you through the
account creation process. After that, Hypergolix will automatically load the
existing account, prompting you only for your Hypergolix password.

.. warning::

    If you want Hypergolix to connect with other computers, you must configure
    remote(s). See above.
    
.. note::

    Hypergolix is always free to use locally, but on the ``hgx.hypergolix.com``
    remote persistence server, accounts are limited to read-only access (10MB
    up, unlimited down) until they register. Registration currently costs
    $10/month.

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

-------------------------------------------------------------------------------
Running a Hypergolix server
-------------------------------------------------------------------------------

Running your own Hypergolix remote persistence server is also easy. It can be
done on a computer that is also running a Hypergolix app daemon. If you want
the daemon to be able to connect to your server on startup, you should start
the server first. When both starting and stopping the server, you must
explicitly pass the path to the PID file.

Starting the server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    # Start like this
    hypergolix start server [-h]
                            pidfile
                            [--cachedir CACHEDIR]
                            [--host HOST]
                            [--port PORT]
                            [--chdir CHDIR]
                            [--logdir LOGDIR]
                            [--debug]
                            [--traceur]
                            [--verbosity {debug,info,warning,error,shouty,extreme}]

-h, --help      Shows the help message and exits
-c, --cachedir  Specify a directory to use as a persistent cache for
                files. If none is specified, will default to an in-
                memory-only cache, which is, quite obviously, rather
                volatile.
-H, --host      Specify the TCP host to use. Defaults to localhost
                only. Passing the special (case-sensitive) string
                "AUTO" will determine the current local IP address and
                bind to that. Passing the special (case-sensitive)
                string "ANY" will bind to any host at the specified
                port (not recommended).
-p, --port      Specify the TCP port to use. Defaults to 7770.
--chdir         Once the daemon starts, chdir it into the specified
                full directory path. By default, the daemon will
                remain in the current directory, which may create
                DirectoryBusy errors.
--logdir        Specify a directory to use for logs. Every service
                failure, error, message, etc will go to dev/null
                without this.
--debug         Enable debug mode. Sets verbosity to debug unless overridden.

-v, --verbosity
                Specify the logging level. Valid options: ``extreme``, 
                ``shouty``, ``louder``, ``loud``, ``normal``, ``quiet``
                
.. note::

    If you are running the service locally, **please enable logging, with a
    verbosity of debug,** and consider enabling debug mode. This will help the
    Hypergolix dev team troubleshoot any problems that arise during operation.

Stopping the server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    
.. code-block:: bash

    hypergolix stop server [-h] pidfile
