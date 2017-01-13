===============================================================================
"Telemeter" remote monitoring example application
===============================================================================

Telemeter is a very basic Python application. It uses ``psutil`` to monitor
system usage, and then pairs with another Hypergolix identity, broadcasting
its system usage on an interval controlled by the remote monitor.

-------------------------------------------------------------------------------
Getting started
-------------------------------------------------------------------------------

After installing Hypergolix, make sure you configure (``hypergolix config
--add hgx``) and run it (``hypergolix start app``) on both the monitoring (from
now on, the "monitor") and monitored (from now on, the "server") computers.
Ideally, also register Hypergolix (``hypergolix config --register``) for the
server, to avoid hitting the storage limit for (read-only) free accounts.

Now, set up a project directory and a virtual environment. Don't forget to
``pip install hgx`` in the environment. I'll use these:

.. code-block:: bash

    mkdir telemeter
    cd telemeter
    python3 -m venv env
    env/bin/pip install hgx
    
.. warning::

    On Windows, replace every ``env/bin/pip`` or ``env/bin/python`` with
    ``env/Scripts/pip`` and ``env/Scripts/python``, respectively.

-------------------------------------------------------------------------------
Hypergolix "Hello world"
-------------------------------------------------------------------------------

To get things started, let's just write a quick application using the
blocking (threadsafe) API. It'll just loop forever, recording a timestamp for
every loop.

To start, we need to create the Hypergolix inter-process communication link, so
we can talk to Hypergolix, and define an interval:

.. code-block:: python

    class Telemeter:
        ''' Remote monitoring demo app.
        '''
        
        def __init__(self, interval):
            self.hgxlink = hgx.HGXLink()
            self.interval = interval

Now, to set up the app, we want to create a Hypergolix object where we'll store
the timestamps. We could make our own serialization system, but Hypergolix
ships with JSON objects available, so let's use those:

.. code-block:: python
            
    def app_init(self):
        ''' Set up the application.
        '''
        self.status_reporter = self.hgxlink.new_threadsafe(
            cls = hgx.JsonObj,
            state = 'Hello world!'
        )
        
And finally, let's make an app loop to continually update the timestamp until
we press ``Ctrl+C``:

.. code-block:: python
            
    def app_run(self):
        ''' Do the main application loop.
        '''
        while True:
            timestamp = datetime.datetime.now().strftime('%Y.%M.%d @ %H:%M:%S')
            self.status_reporter.state = timestamp
            self.status_reporter.push_threadsafe()
            print('Logged ' + timestamp)
            time.sleep(self.interval)
            
That's all that we need for "Hello world"! Putting it all together, and adding
an entry point so we can invoke the script as ``env/bin/python telemeter.py``:

.. code-block:: python

    import time
    import datetime
    import hgx


    class Telemeter:
        ''' Remote monitoring demo app.
        '''
        
        def __init__(self, interval):
            self.hgxlink = hgx.HGXLink()
            self.interval = interval
            
            # These are the actual Hypergolix business parts
            self.status_reporter = None
            
        def app_init(self):
            ''' Set up the application.
            '''
            self.status_reporter = self.hgxlink.new_threadsafe(
                cls = hgx.JsonObj,
                state = 'Hello world!'
            )
            print('Created status object: ' + self.status_reporter.ghid.as_str())
            
        def app_run(self):
            ''' Do the main application loop.
            '''
            while True:
                timestamp = datetime.datetime.now().strftime('%Y.%m.%d @ %H:%M:%S')
                self.status_reporter.state = timestamp
                self.status_reporter.push_threadsafe()
                print('Logged ' + timestamp)
                time.sleep(self.interval)


    if __name__ == "__main__":
        try:
            app = Telemeter(interval=5)
            app.app_init()
            app.app_run()
            
        finally:
            app.hgxlink.stop_threadsafe()
            
Great! Now we have a really simple Hypergolix app. But at the moment, it's not
particularly useful -- sure, we're logging timestamps, but nobody can see them.
Though, if you're feeling particularly adventurous, you could open up a Python
interpreter and manually retrieve the status like this:

.. code-block:: python

    >>> import hgx
    >>> hgxlink = hgx.HGXLink()
    >>> # Make sure to replace this with the "Created status object: <GHID>"
    >>> ghid = hgx.Ghid.from_str('<GHID>')
    >>> status_reporter = hgxlink.get_threadsafe(cls=hgx.JsonObj, ghid=ghid)
    >>> status_reporter.state
    '2016.12.14 @ 09:17:10'
    >>> # Wait 5 seconds and...
    >>> status_reporter.state
    '2016.12.14 @ 09:17:15'
    
If you keep calling ``status_reporter.state``, you'll see the timestamp
automatically update. Neat! But, we want to do a little more...

-------------------------------------------------------------------------------
A bugfix, plus pairing
-------------------------------------------------------------------------------

We want Telemeter to talk to another computer. To do that, we need to register
a share handler. Share handlers tell Hypergolix that an application is
available to handle specific kinds of objects from other Hypergolix accounts.
But first, if you watched ``stdout`` closely in the last step, you might have
seen a bug:

.. code-block:: none

    Logged 2016.12.14 @ 09:17:10
    Logged 2016.12.14 @ 09:17:15
    Logged 2016.12.14 @ 09:17:20
    Logged 2016.12.14 @ 09:17:25
    Logged 2016.12.14 @ 09:17:31
    Logged 2016.12.14 @ 09:17:36
    Logged 2016.12.14 @ 09:17:41
    Logged 2016.12.14 @ 09:17:46
    Logged 2016.12.14 @ 09:17:52

Notice how the clock is wandering? The :meth:`Obj.push_threadsafe()` takes some
time -- it needs to talk to the Hypergolix server. A permanent solution might
use a generator to constantly generate intervals based on the initial time, but
a quick and dirty solution is just to change the ``time.sleep`` call to
compensate for the delay:

.. code-block:: python
        
    def app_run(self):
        ''' Do the main application loop.
        '''
        while True:
            timestamp = datetime.datetime.now()
            timestr = timestamp.strftime('%Y.%m.%d @ %H:%M:%S')
            
            self.status.state = timestr
            self.status.push_threadsafe()
            
            elapsed = (datetime.datetime.now() - timestamp).total_seconds()
            print('Logged {0} in {1:.3f} seconds.'.format(timestr, elapsed))
            time.sleep(self.interval - elapsed)
            
With that sorted, we can start working on pairing. Thinking a bit about how we
want the app to work, we'd like the server to automatically log its status,
and for some other computer to occasionally check in on it. But we don't want
anyone and everyone to have access to our server's CPU status! So as a quick
approximation, let's set up a trust-on-first-connect construct: the first
account that connects to the server can watch its status, but any subsequent
account cannot.

But first, the server needs to know that the monitor is trying to connect. So
we'll define a dedicated pairing object: a small, special object that the
monitor can send the server, to request the server's status. To do that, we'll
create a specific pairing ``API ID``.

Hypergolix uses ``API ID``\ s as a kind of schema identifier for objects. Their
meaning is application-specific, but in general you should generate a random
API ID using ``hgx.utils.ApiID.pseudorandom()`` to avoid accidental collisions
with other applications. ``API ID``\ s are used in three ways:

1.  In general, to explicitly define the object's format and/or purpose
2.  For Hypergolix, to dispatch shared objects to applications that have
    registered share handlers for them
3.  For applications, to handle the actual objects

To pair, we're first going to generate (and then, in this case, hard-code) a
random ``API ID``. We'll use this to identify objects whose sole purpose is for
the monitor to announce its existence to the server:

.. code-block:: python

    PAIR_API = hgx.utils.ApiID(
        b'\x17\n\x12\x17\x03\x0f\x14\x11\x07\x10\x05\x04' +
        b'\x14\x18\x11\x11\x12\x02\x17\x12\x15\x0e\x04' +
        b'\x0f\x11\x19\x07\x19\n\r\x03\x06\x12\x04\x17' +
        b'\x11\x14\x07\t\x08\x13\x19\x04\n\x0f\x15\x12' +
        b'\x14\x07\x19\x16\x13\x18\x0b\x18\x0e\x12\x15\n' +
        b'\n\x16\x0f\x08\x14'
    )

Now, on the server application, we're going to register a share handler for
that ``API ID``:

.. code-block:: python
        
    def pair_handler(self, ghid, origin, api_id):
        ''' Pair handlers ignore the object itself, instead setting up
        the origin as the paired_fingerprint (unless one already exists,
        in which case it is ignored) and sharing the status object with
        them.
        
        This also doubles as a way to re-pair the same fingerprint, in
        the event that they have gone offline for a long time and are no
        longer receiving updates.
        '''
        # The initial pairing (pair/trust on first connect)
        if self.paired_fingerprint is None:
            self.paired_fingerprint = origin
        
        # Subsequent pairing requests from anyone else are ignored
        elif self.paired_fingerprint != origin:
            return
            
        # Now we want to share the status reporter, if we have one, with the
        # origin
        if self.status_reporter is not None:
            self.status_reporter.share_threadsafe(origin)
            
Share handlers are invoked with the :class:`Ghid` ``ghid`` of the object being
shared, the :class:`Ghid` ``origin`` of the account that shared it, and the
:class:`hgx.utils.ApiID` ``api_id`` of the object itself. So when our server
gets a shared object with the correct ``API ID``, it will check to see if it
already has a monitor, and, if so, if the pair request is coming from the
existing handler (that's the "trust on first connect" bit). If someone else
tries to pair, the handler returns immediately, doing nothing. Otherwise, it
shares the status object with the monitor.

Now, before we register the share handler (``pair_handler``) with the
:class:`HGXLink`, we need to wrap the handler so that the link's internal event
loop can ``await`` it:

.. code-block:: python
        
    # Share handlers are called from within the HGXLink event loop, so they
    # must be wrapped before use
    pair_handler = self.hgxlink.wrap_threadsafe(self.pair_handler)
    self.hgxlink.register_share_handler_threadsafe(PAIR_API, pair_handler)

Now the server is set up to pair with the monitor, though the monitor can't do
anything yet. Putting it all together:

.. code-block:: python
    
    import time
    import datetime
    import hgx


    # These are app-specific (here, totally random) API schema identifiers
    STATUS_API = hgx.utils.ApiID(
        b'\x02\x0b\x16\x19\x00\x19\x10\x18\x08\x12\x03' +
        b'\x11\x07\x07\r\x0c\n\x14\x04\x13\x07\x04\x06' +
        b'\x13\x01\x0c\x04\x00\x0b\x03\x01\x12\x05\x0f' +
        b'\x01\x0c\x05\x11\x03\x01\x0e\x13\x16\x13\x11' +
        b'\x10\x13\t\x06\x10\x00\x14\x0c\x15\x0b\x07' +
        b'\x0c\x0c\x04\x07\x0b\x0f\x18\x03'
    )
    PAIR_API = hgx.utils.ApiID(
        b'\x17\n\x12\x17\x03\x0f\x14\x11\x07\x10\x05\x04' +
        b'\x14\x18\x11\x11\x12\x02\x17\x12\x15\x0e\x04' +
        b'\x0f\x11\x19\x07\x19\n\r\x03\x06\x12\x04\x17' +
        b'\x11\x14\x07\t\x08\x13\x19\x04\n\x0f\x15\x12' +
        b'\x14\x07\x19\x16\x13\x18\x0b\x18\x0e\x12\x15\n' +
        b'\n\x16\x0f\x08\x14'
    )


    class Telemeter:
        ''' Remote monitoring demo app sender.
        '''
        
        def __init__(self, interval):
            self.hgxlink = hgx.HGXLink()
            self.interval = interval
            
            # These are the actual Hypergolix business parts
            self.status = None
            self.paired_fingerprint = None
            
        def app_init(self):
            ''' Set up the application.
            '''
            self.status = self.hgxlink.new_threadsafe(
                cls = hgx.JsonObj,
                state = 'Hello world!',
                api_id = STATUS_API
            )
            
            # Share handlers are called from within the HGXLink event loop, so they
            # must be wrapped before use
            pair_handler = self.hgxlink.wrap_threadsafe(self.pair_handler)
            self.hgxlink.register_share_handler_threadsafe(PAIR_API, pair_handler)
            
        def app_run(self):
            ''' Do the main application loop.
            '''
            while True:
                timestamp = datetime.datetime.now()
                timestr = timestamp.strftime('%Y.%m.%d @ %H:%M:%S')
                
                self.status.state = timestr
                self.status.push_threadsafe()
                
                elapsed = (datetime.datetime.now() - timestamp).total_seconds()
                print('Logged {0} in {1:.3f} seconds.'.format(timestr, elapsed))
                time.sleep(self.interval - elapsed)
            
        def pair_handler(self, ghid, origin, api_id):
            ''' Pair handlers ignore the object itself, instead setting up
            the origin as the paired_fingerprint (unless one already exists,
            in which case it is ignored) and sharing the status object with
            them.
            
            This also doubles as a way to re-pair the same fingerprint, in
            the event that they have gone offline for a long time and are no
            longer receiving updates.
            '''
            # The initial pairing (pair/trust on first connect)
            if self.paired_fingerprint is None:
                self.paired_fingerprint = origin
            
            # Subsequent pairing requests from anyone else are ignored
            elif self.paired_fingerprint != origin:
                return
                
            # Now we want to share the status reporter, if we have one, with the
            # origin
            if self.status_reporter is not None:
                self.status_reporter.share_threadsafe(origin)


    if __name__ == "__main__":
        try:
            app = Telemeter(interval=5)
            app.app_init()
            app.app_run()
            
        finally:
            app.hgxlink.stop_threadsafe()

-------------------------------------------------------------------------------
Pairing, client-side
-------------------------------------------------------------------------------

Status check: the server is ready to broadcast timestamps to the monitor, but
the monitor doesn't know how to request, nor receive them. So we'll create a
monitor object that creates a pair request on startup:

.. code-block:: python

    class Monitor:
        ''' Remote monitoring demo app receiver.
        '''
        
        def __init__(self, telemeter_fingerprint):
            self.hgxlink = hgx.HGXLink()
            self.telemeter_fingerprint = telemeter_fingerprint
            
            # These are the actual Hypergolix business parts
            self.status = None
            self.pair = None
            
        def app_init(self):
            ''' Set up the application.
            '''            
            # Wait until after registering the share handler to avoid a race
            # condition with the Telemeter
            self.pair = self.hgxlink.new_threadsafe(
                cls = hgx.JsonObj,
                state = 'Hello world!',
                api_id = PAIR_API
            )
            self.pair.share_threadsafe(self.telemeter_fingerprint)
            
With that, the Monitor can request the server's status. Thus far, our app's
logic flow looks like this:

1.  Start server telemeter
2.  Server logs timestamps, waiting for pairing request
3.  Monitor sends pairing request
4.  Server responds with the timestamp object

But, the monitor doesn't know how to do anything with the server's timestamp
object yet, so let's revisit the monitor. This time around, we'll make use of
the :class:`HGXLink`\ 's native async API for the share handler to make the
code a little cleaner:

.. code-block:: python
        
    async def status_handler(self, ghid, origin, api_id):
        ''' We sent the pairing, and the Telemeter shared its status obj
        with us in return. Get it, store it locally, and register a
        callback to run every time the object is updated.
        '''
        status = await self.hgxlink.get(
            cls = hgx.JsonObj,
            ghid = ghid
        )
        # This registers the update callback. It will be run in the hgxlink
        # event loop, so if it were blocking/threaded, we would need to wrap
        # it like this: self.hgxlink.wrap_threadsafe(self.update_handler)
        status.callback = self.update_handler
        # We're really only doing this to prevent garbage collection
        self.status = status
        
As before, we need to handle the incoming object's address, origin, and
``API ID``. But this time, we want to actually do something with the object:
we'll store it locally under ``self.status``, and then we register the
following simple callback to run every time Hypergolix gets an update for it:

.. code-block:: python
        
    async def update_handler(self, obj):
        ''' A very simple, **asynchronous** handler for status updates.
        This will be called every time the Telemeter changes their
        status.
        '''
        print(obj.state)
        
For simplicity's sake, we'll add a busy-wait loop for the monitor, and a small
``argparser`` to switch between the server "telemeter" and the monitor
"telemeter". Don't forget to actually register the share handler (look in
``Monitor.app_init``), and then we're good to go! Summing up:

.. code-block:: python

    import argparse
    import time
    import datetime
    import hgx


    # These are app-specific (here, totally random) API schema identifiers
    STATUS_API = hgx.utils.ApiID(
        b'\x02\x0b\x16\x19\x00\x19\x10\x18\x08\x12\x03' +
        b'\x11\x07\x07\r\x0c\n\x14\x04\x13\x07\x04\x06' +
        b'\x13\x01\x0c\x04\x00\x0b\x03\x01\x12\x05\x0f' +
        b'\x01\x0c\x05\x11\x03\x01\x0e\x13\x16\x13\x11' +
        b'\x10\x13\t\x06\x10\x00\x14\x0c\x15\x0b\x07' +
        b'\x0c\x0c\x04\x07\x0b\x0f\x18\x03'
    )
    PAIR_API = hgx.utils.ApiID(
        b'\x17\n\x12\x17\x03\x0f\x14\x11\x07\x10\x05\x04' +
        b'\x14\x18\x11\x11\x12\x02\x17\x12\x15\x0e\x04' +
        b'\x0f\x11\x19\x07\x19\n\r\x03\x06\x12\x04\x17' +
        b'\x11\x14\x07\t\x08\x13\x19\x04\n\x0f\x15\x12' +
        b'\x14\x07\x19\x16\x13\x18\x0b\x18\x0e\x12\x15\n' +
        b'\n\x16\x0f\x08\x14'
    )


    class Telemeter:
        ''' Remote monitoring demo app sender.
        '''
        
        def __init__(self, interval):
            self.hgxlink = hgx.HGXLink()
            self.interval = interval
            
            # These are the actual Hypergolix business parts
            self.status = None
            self.paired_fingerprint = None
            
        def app_init(self):
            ''' Set up the application.
            '''
            print('My fingerprint is: ' + self.hgxlink.whoami.as_str())
            self.status = self.hgxlink.new_threadsafe(
                cls = hgx.JsonObj,
                state = 'Hello world!',
                api_id = STATUS_API
            )
            
            # Share handlers are called from within the HGXLink event loop, so they
            # must be wrapped before use
            pair_handler = self.hgxlink.wrap_threadsafe(self.pair_handler)
            self.hgxlink.register_share_handler_threadsafe(PAIR_API, pair_handler)
            
        def app_run(self):
            ''' Do the main application loop.
            '''
            while True:
                timestamp = datetime.datetime.now()
                timestr = timestamp.strftime('%Y.%m.%d @ %H:%M:%S')
                
                self.status.state = timestr
                self.status.push_threadsafe()
                
                elapsed = (datetime.datetime.now() - timestamp).total_seconds()
                print('Logged {0} in {1:.3f} seconds.'.format(timestr, elapsed))
                time.sleep(self.interval - elapsed)
            
        def pair_handler(self, ghid, origin, api_id):
            ''' Pair handlers ignore the object itself, instead setting up
            the origin as the paired_fingerprint (unless one already exists,
            in which case it is ignored) and sharing the status object with
            them.
            
            This also doubles as a way to re-pair the same fingerprint, in
            the event that they have gone offline for a long time and are no
            longer receiving updates.
            '''
            # The initial pairing (pair/trust on first connect)
            if self.paired_fingerprint is None:
                self.paired_fingerprint = origin
            
            # Subsequent pairing requests from anyone else are ignored
            elif self.paired_fingerprint != origin:
                return
                
            # Now we want to share the status reporter, if we have one, with the
            # origin
            if self.status is not None:
                self.status.share_threadsafe(origin)


    class Monitor:
        ''' Remote monitoring demo app receiver.
        '''
        
        def __init__(self, telemeter_fingerprint):
            self.hgxlink = hgx.HGXLink()
            self.telemeter_fingerprint = telemeter_fingerprint
            
            # These are the actual Hypergolix business parts
            self.status = None
            self.pair = None
            
        def app_init(self):
            ''' Set up the application.
            '''
            # Because we're using a native coroutine for this share handler, it
            # needs no wrapping.
            self.hgxlink.register_share_handler_threadsafe(STATUS_API,
                                                           self.status_handler)
            
            # Wait until after registering the share handler to avoid a race
            # condition with the Telemeter
            self.pair = self.hgxlink.new_threadsafe(
                cls = hgx.JsonObj,
                state = 'Hello world!',
                api_id = PAIR_API
            )
            self.pair.share_threadsafe(self.telemeter_fingerprint)
            
        async def status_handler(self, ghid, origin, api_id):
            ''' We sent the pairing, and the Telemeter shared its status obj
            with us in return. Get it, store it locally, and register a
            callback to run every time the object is updated.
            '''
            status = await self.hgxlink.get(
                cls = hgx.JsonObj,
                ghid = ghid
            )
            # This registers the update callback. It will be run in the hgxlink
            # event loop, so if it were blocking/threaded, we would need to wrap
            # it like this: self.hgxlink.wrap_threadsafe(self.update_handler)
            status.callback = self.update_handler
            # We're really only doing this to prevent garbage collection
            self.status = status
            
        async def update_handler(self, obj):
            ''' A very simple, **asynchronous** handler for status updates.
            This will be called every time the Telemeter changes their
            status.
            '''
            print(obj.state)
            
        def app_run(self):
            ''' For now, just busy-wait.
            '''
            while True:
                time.sleep(1)


    if __name__ == "__main__":
        argparser = argparse.ArgumentParser(
            description = 'A simple remote telemetry app.'
        )
        argparser.add_argument(
            '--telereader',
            action = 'store',
            default = None,
            help = 'Pass a Telemeter fingerprint to run as a reader.'
        )
        args = argparser.parse_args()
        
        if args.telereader is not None:
            telemeter_fingerprint = hgx.Ghid.from_str(args.telereader)
            app = Monitor(telemeter_fingerprint)
            
        else:
            app = Telemeter(interval=5)
            
        try:
            app.app_init()
            app.app_run()
            
        finally:
            app.hgxlink.stop_threadsafe()

-------------------------------------------------------------------------------
Pairing, client-side
-------------------------------------------------------------------------------

Now that we've got the server and the monitor talking, it's time to make them
actually do something worthwhile. First, let's make the logging interval
adjustable in the ``Telemeter``:

.. code-block:: python

    INTERVAL_API = hgx.utils.ApiID(
        b'\n\x10\x04\x00\x13\x11\x0b\x11\x06\x02\x19\x00' +
        b'\x11\x12\x10\x10\n\x14\x19\x15\x11\x18\x0f\x0f' +
        b'\x01\r\x0c\x15\x16\x04\x0f\x18\x19\x13\x14\x11' +
        b'\x10\x01\x19\x19\x15\x0b\t\x0e\x15\r\x16\x15' +
        b'\x0e\n\x19\x0b\x14\r\n\x04\x0c\x06\x03\x13\x01' +
        b'\x01\x12\x05'
    )
            
    def interval_handler(self, ghid, origin, api_id):
        ''' Interval handlers change our recording interval.
        '''
        # Ignore requests that don't match our pairing.
        # This will also catch un-paired requests.
        if origin != self.paired_fingerprint:
            return
        
        # If the address matches our pairing, use it to change our interval.
        else:
            # We don't need to create an update callback here, because any
            # upstream modifications will automatically be passed to the
            # object. This is true of all hypergolix objects, but by using a
            # proxy, it mimics the behavior of the int itself.
            interval_proxy = self.hgxlink.get_threadsafe(
                cls = hgx.JsonProxy,
                ghid = ghid
            )
            self._interval = interval_proxy
            
    @property
    def interval(self):
        ''' This provides some consumer-side protection against
        malicious interval proxies.
        '''
        try:
            return float(max(self._interval, self.minimum_interval))
        
        except (ValueError, TypeError):
            return self.minimum_interval

And now let's add some code to the ``Monitor`` to adjust the interval remotely:

.. code-block:: python
        
    def set_interval(self, interval):
        ''' Set the recording interval remotely.
        '''
        # This is some supply-side protection of the interval.
        interval = float(interval)
        
        if self.interval is None:
            self.interval = self.hgxlink.new_threadsafe(
                cls = hgx.JsonProxy,
                state = interval,
                api_id = INTERVAL_API
            )
            self.interval.hgx_share_threadsafe(self.telemeter_fingerprint)
        else:
            # We can't directly reassign the proxy here, because it would just
            # overwrite the self.interval name with the interval float from
            # above. Instead, we need to assign to the state.
            self.interval.hgx_state = interval
            self.interval.hgx_push_threadsafe()
            
Now for a status check. We should be able to run the telemeter and adjust the
interval remotely:

.. code-block:: python

    import argparse
    import time
    import datetime
    import hgx


    # These are app-specific (here, totally random) API schema identifiers
    STATUS_API = hgx.utils.ApiID(
        b'\x02\x0b\x16\x19\x00\x19\x10\x18\x08\x12\x03' +
        b'\x11\x07\x07\r\x0c\n\x14\x04\x13\x07\x04\x06' +
        b'\x13\x01\x0c\x04\x00\x0b\x03\x01\x12\x05\x0f' +
        b'\x01\x0c\x05\x11\x03\x01\x0e\x13\x16\x13\x11' +
        b'\x10\x13\t\x06\x10\x00\x14\x0c\x15\x0b\x07' +
        b'\x0c\x0c\x04\x07\x0b\x0f\x18\x03'
    )
    PAIR_API = hgx.utils.ApiID(
        b'\x17\n\x12\x17\x03\x0f\x14\x11\x07\x10\x05\x04' +
        b'\x14\x18\x11\x11\x12\x02\x17\x12\x15\x0e\x04' +
        b'\x0f\x11\x19\x07\x19\n\r\x03\x06\x12\x04\x17' +
        b'\x11\x14\x07\t\x08\x13\x19\x04\n\x0f\x15\x12' +
        b'\x14\x07\x19\x16\x13\x18\x0b\x18\x0e\x12\x15\n' +
        b'\n\x16\x0f\x08\x14'
    )
    INTERVAL_API = hgx.utils.ApiID(
        b'\n\x10\x04\x00\x13\x11\x0b\x11\x06\x02\x19\x00' +
        b'\x11\x12\x10\x10\n\x14\x19\x15\x11\x18\x0f\x0f' +
        b'\x01\r\x0c\x15\x16\x04\x0f\x18\x19\x13\x14\x11' +
        b'\x10\x01\x19\x19\x15\x0b\t\x0e\x15\r\x16\x15' +
        b'\x0e\n\x19\x0b\x14\r\n\x04\x0c\x06\x03\x13\x01' +
        b'\x01\x12\x05'
    )


    class Telemeter:
        ''' Remote monitoring demo app sender.
        '''
        
        def __init__(self, interval, minimum_interval=1):
            self.hgxlink = hgx.HGXLink()
            self._interval = interval
            self.minimum_interval = minimum_interval
            
            # These are the actual Hypergolix business parts
            self.status = None
            self.paired_fingerprint = None
            
        def app_init(self):
            ''' Set up the application.
            '''
            print('My fingerprint is: ' + self.hgxlink.whoami.as_str())
            self.status = self.hgxlink.new_threadsafe(
                cls = hgx.JsonObj,
                state = 'Hello world!',
                api_id = STATUS_API
            )
            
            # Share handlers are called from within the HGXLink event loop, so they
            # must be wrapped before use
            pair_handler = self.hgxlink.wrap_threadsafe(self.pair_handler)
            self.hgxlink.register_share_handler_threadsafe(PAIR_API, pair_handler)
            # And set up a handler to change our interval
            interval_handler = self.hgxlink.wrap_threadsafe(self.interval_handler)
            self.hgxlink.register_share_handler_threadsafe(INTERVAL_API,
                                                           interval_handler)
            
        def app_run(self):
            ''' Do the main application loop.
            '''
            while True:
                timestamp = datetime.datetime.now()
                timestr = timestamp.strftime('%Y.%m.%d @ %H:%M:%S')
                
                self.status.state = timestr
                self.status.push_threadsafe()
                
                elapsed = (datetime.datetime.now() - timestamp).total_seconds()
                print('Logged {0} in {1:.3f} seconds.'.format(timestr, elapsed))
                # Make sure we clamp this to non-negative values, in case the
                # update took longer than the current interval.
                time.sleep(max(self.interval - elapsed, 0))
            
        def pair_handler(self, ghid, origin, api_id):
            ''' Pair handlers ignore the object itself, instead setting up
            the origin as the paired_fingerprint (unless one already exists,
            in which case it is ignored) and sharing the status object with
            them.
            
            This also doubles as a way to re-pair the same fingerprint, in
            the event that they have gone offline for a long time and are no
            longer receiving updates.
            '''
            # The initial pairing (pair/trust on first connect)
            if self.paired_fingerprint is None:
                self.paired_fingerprint = origin
            
            # Subsequent pairing requests from anyone else are ignored
            elif self.paired_fingerprint != origin:
                return
                
            # Now we want to share the status reporter, if we have one, with the
            # origin
            if self.status is not None:
                self.status.share_threadsafe(origin)
                
        def interval_handler(self, ghid, origin, api_id):
            ''' Interval handlers change our recording interval.
            '''
            # Ignore requests that don't match our pairing.
            # This will also catch un-paired requests.
            if origin != self.paired_fingerprint:
                return
            
            # If the address matches our pairing, use it to change our interval.
            else:
                # We don't need to create an update callback here, because any
                # upstream modifications will automatically be passed to the
                # object. This is true of all hypergolix objects, but by using a
                # proxy, it mimics the behavior of the int itself.
                interval_proxy = self.hgxlink.get_threadsafe(
                    cls = hgx.JsonProxy,
                    ghid = ghid
                )
                self._interval = interval_proxy
                
        @property
        def interval(self):
            ''' This provides some consumer-side protection against
            malicious interval proxies.
            '''
            try:
                return float(max(self._interval, self.minimum_interval))
            
            except (ValueError, TypeError):
                return self.minimum_interval


    class Monitor:
        ''' Remote monitoring demo app receiver.
        '''
        
        def __init__(self, telemeter_fingerprint):
            self.hgxlink = hgx.HGXLink()
            self.telemeter_fingerprint = telemeter_fingerprint
            
            # These are the actual Hypergolix business parts
            self.status = None
            self.pair = None
            self.interval = None
            
        def app_init(self):
            ''' Set up the application.
            '''
            # Because we're using a native coroutine for this share handler, it
            # needs no wrapping.
            self.hgxlink.register_share_handler_threadsafe(STATUS_API,
                                                           self.status_handler)
            
            # Wait until after registering the share handler to avoid a race
            # condition with the Telemeter
            self.pair = self.hgxlink.new_threadsafe(
                cls = hgx.JsonObj,
                state = 'Hello world!',
                api_id = PAIR_API
            )
            self.pair.share_threadsafe(self.telemeter_fingerprint)
            
        def app_run(self):
            ''' For now, just busy-wait.
            '''
            while True:
                time.sleep(1)
            
        async def status_handler(self, ghid, origin, api_id):
            ''' We sent the pairing, and the Telemeter shared its status obj
            with us in return. Get it, store it locally, and register a
            callback to run every time the object is updated.
            '''
            status = await self.hgxlink.get(
                cls = hgx.JsonObj,
                ghid = ghid
            )
            # This registers the update callback. It will be run in the hgxlink
            # event loop, so if it were blocking/threaded, we would need to wrap
            # it like this: self.hgxlink.wrap_threadsafe(self.update_handler)
            status.callback = self.update_handler
            # We're really only doing this to prevent garbage collection
            self.status = status
            
        async def update_handler(self, obj):
            ''' A very simple, **asynchronous** handler for status updates.
            This will be called every time the Telemeter changes their
            status.
            '''
            print(obj.state)
            
        def set_interval(self, interval):
            ''' Set the recording interval remotely.
            '''
            # This is some supply-side protection of the interval.
            interval = float(interval)
            
            if self.interval is None:
                self.interval = self.hgxlink.new_threadsafe(
                    cls = hgx.JsonProxy,
                    state = interval,
                    api_id = INTERVAL_API
                )
                self.interval.hgx_share_threadsafe(self.telemeter_fingerprint)
            else:
                # We can't directly reassign the proxy here, because it would just
                # overwrite the self.interval name with the interval float from
                # above. Instead, we need to assign to the state.
                self.interval.hgx_state = interval
                self.interval.hgx_push_threadsafe()


    if __name__ == "__main__":
        argparser = argparse.ArgumentParser(
            description = 'A simple remote telemetry app.'
        )
        argparser.add_argument(
            '--telereader',
            action = 'store',
            default = None,
            help = 'Pass a Telemeter fingerprint to run as a reader.'
        )
        argparser.add_argument(
            '--interval',
            action = 'store',
            default = None,
            type = float,
            help = 'Set the Telemeter recording interval from the Telereader. ' +
                   'Ignored by a Telemeter.'
        )
        args = argparser.parse_args()
        
        if args.telereader is not None:
            telemeter_fingerprint = hgx.Ghid.from_str(args.telereader)
            app = Monitor(telemeter_fingerprint)
            
            try:
                app.app_init()
                
                if args.interval is not None:
                    app.set_interval(args.interval)
                
                app.app_run()
                
            finally:
                app.hgxlink.stop_threadsafe()
            
        else:
            app = Telemeter(interval=5)
                
            try:
                app.app_init()
                app.app_run()
                
            finally:
                app.hgxlink.stop_threadsafe()

-------------------------------------------------------------------------------
Enter ``psutil``
-------------------------------------------------------------------------------

We've got a simple, adjustable-interval timestamp program running between the
monitor and the server. Now let's make the server actually *monitor* something.
For this, we'll use ``psutil``, a cross-platform system monitoring library.

.. sidebar::

    Make sure to ``pip install psutil`` into the environment!
    
First we're going to make some quick utilities for formatting purposes. These
will make our server logs much easier to read:

.. code-block:: python

    def humanize_bibytes(n, prefixes=collections.OrderedDict((
                        (0, 'B'),
                        (1024, 'KiB'),
                        (1048576, 'MiB'),
                        (1073741824, 'GiB'),
                        (1099511627776, 'TiB'),
                        (1125899906842624, 'PiB'),
                        (1152921504606846976, 'EiB'),
                        (1180591620717411303424, 'ZiB'),
                        (1208925819614629174706176, 'YiB')))):
        ''' Convert big numbers into easily-human-readable ones.
        '''
        for value, prefix in reversed(prefixes.items()):
            if n >= value:
                return '{:.2f} {}'.format(float(n) / value, prefix)
                
                
    def format_cpu(cpu_list):
        cpustr = 'CPU:\n----------\n'
        for cpu in cpu_list:
            cpustr += '  ' + str(cpu) + '%\n'
        return cpustr
        
        
    def format_mem(mem_tup):
        memstr = 'MEM:\n----------\n'
        memstr += '  Avail: ' + humanize_bibytes(mem_tup.available) + '\n'
        memstr += '  Total: ' + humanize_bibytes(mem_tup.total) + '\n'
        memstr += '  Used:  ' + str(mem_tup.percent) + '%\n'
        return memstr
        
        
    def format_disk(disk_tup):
        diskstr = 'DISK:\n----------\n'
        diskstr += '  Avail: ' + humanize_bibytes(disk_tup.free) + '\n'
        diskstr += '  Total: ' + humanize_bibytes(disk_tup.total) + '\n'
        diskstr += '  Used:  ' + str(disk_tup.percent) + '%\n'
        return diskstr

Great. Now we just need to *slightly* modify the ``Telemeter.app_run`` method
to send our system usage instead of just the timestamp:

.. code-block:: python
        
    def app_run(self):
        ''' Do the main application loop.
        '''
        while True:
            timestamp = datetime.datetime.now()
            timestr = timestamp.strftime('%Y.%m.%d @ %H:%M:%S\n==========\n')
            cpustr = format_cpu(psutil.cpu_percent(interval=.1, percpu=True))
            memstr = format_mem(psutil.virtual_memory())
            diskstr = format_disk(psutil.disk_usage('/'))
            
            status = (timestr + cpustr + memstr + diskstr + '\n')
            
            self.status.state = status
            self.status.push_threadsafe()
            
            elapsed = (datetime.datetime.now() - timestamp).total_seconds()
            print('Logged in {:.3f} seconds:\n{}'.format(elapsed, status))
            # Make sure we clamp this to non-negative values, in case the
            # update took longer than the current interval.
            time.sleep(max(self.interval - elapsed, 0))
            
All together now...

.. code-block:: python

    import argparse
    import time
    import datetime
    import psutil
    import collections
    import hgx


    # These are app-specific (here, totally random) API schema identifiers
    STATUS_API = hgx.utils.ApiID(
        b'\x02\x0b\x16\x19\x00\x19\x10\x18\x08\x12\x03' +
        b'\x11\x07\x07\r\x0c\n\x14\x04\x13\x07\x04\x06' +
        b'\x13\x01\x0c\x04\x00\x0b\x03\x01\x12\x05\x0f' +
        b'\x01\x0c\x05\x11\x03\x01\x0e\x13\x16\x13\x11' +
        b'\x10\x13\t\x06\x10\x00\x14\x0c\x15\x0b\x07' +
        b'\x0c\x0c\x04\x07\x0b\x0f\x18\x03'
    )
    PAIR_API = hgx.utils.ApiID(
        b'\x17\n\x12\x17\x03\x0f\x14\x11\x07\x10\x05\x04' +
        b'\x14\x18\x11\x11\x12\x02\x17\x12\x15\x0e\x04' +
        b'\x0f\x11\x19\x07\x19\n\r\x03\x06\x12\x04\x17' +
        b'\x11\x14\x07\t\x08\x13\x19\x04\n\x0f\x15\x12' +
        b'\x14\x07\x19\x16\x13\x18\x0b\x18\x0e\x12\x15\n' +
        b'\n\x16\x0f\x08\x14'
    )
    INTERVAL_API = hgx.utils.ApiID(
        b'\n\x10\x04\x00\x13\x11\x0b\x11\x06\x02\x19\x00' +
        b'\x11\x12\x10\x10\n\x14\x19\x15\x11\x18\x0f\x0f' +
        b'\x01\r\x0c\x15\x16\x04\x0f\x18\x19\x13\x14\x11' +
        b'\x10\x01\x19\x19\x15\x0b\t\x0e\x15\r\x16\x15' +
        b'\x0e\n\x19\x0b\x14\r\n\x04\x0c\x06\x03\x13\x01' +
        b'\x01\x12\x05'
    )


    def humanize_bibytes(n, prefixes=collections.OrderedDict((
                        (0, 'B'),
                        (1024, 'KiB'),
                        (1048576, 'MiB'),
                        (1073741824, 'GiB'),
                        (1099511627776, 'TiB'),
                        (1125899906842624, 'PiB'),
                        (1152921504606846976, 'EiB'),
                        (1180591620717411303424, 'ZiB'),
                        (1208925819614629174706176, 'YiB')))):
        ''' Convert big numbers into easily-human-readable ones.
        '''
        for value, prefix in reversed(prefixes.items()):
            if n >= value:
                return '{:.2f} {}'.format(float(n) / value, prefix)
                
                
    def format_cpu(cpu_list):
        cpustr = 'CPU:\n----------\n'
        for cpu in cpu_list:
            cpustr += '  ' + str(cpu) + '%\n'
        return cpustr
        
        
    def format_mem(mem_tup):
        memstr = 'MEM:\n----------\n'
        memstr += '  Avail: ' + humanize_bibytes(mem_tup.available) + '\n'
        memstr += '  Total: ' + humanize_bibytes(mem_tup.total) + '\n'
        memstr += '  Used:  ' + str(mem_tup.percent) + '%\n'
        return memstr
        
        
    def format_disk(disk_tup):
        diskstr = 'DISK:\n----------\n'
        diskstr += '  Avail: ' + humanize_bibytes(disk_tup.free) + '\n'
        diskstr += '  Total: ' + humanize_bibytes(disk_tup.total) + '\n'
        diskstr += '  Used:  ' + str(disk_tup.percent) + '%\n'
        return diskstr


    class Telemeter:
        ''' Remote monitoring demo app sender.
        '''
        
        def __init__(self, interval, minimum_interval=1):
            self.hgxlink = hgx.HGXLink()
            self._interval = interval
            self.minimum_interval = minimum_interval
            
            # These are the actual Hypergolix business parts
            self.status = None
            self.paired_fingerprint = None
            
        def app_init(self):
            ''' Set up the application.
            '''
            print('My fingerprint is: ' + self.hgxlink.whoami.as_str())
            self.status = self.hgxlink.new_threadsafe(
                cls = hgx.JsonObj,
                state = 'Hello world!',
                api_id = STATUS_API
            )
            
            # Share handlers are called from within the HGXLink event loop, so they
            # must be wrapped before use
            pair_handler = self.hgxlink.wrap_threadsafe(self.pair_handler)
            self.hgxlink.register_share_handler_threadsafe(PAIR_API, pair_handler)
            # And set up a handler to change our interval
            interval_handler = self.hgxlink.wrap_threadsafe(self.interval_handler)
            self.hgxlink.register_share_handler_threadsafe(INTERVAL_API,
                                                           interval_handler)
            
        def app_run(self):
            ''' Do the main application loop.
            '''
            while True:
                timestamp = datetime.datetime.now()
                timestr = timestamp.strftime('%Y.%m.%d @ %H:%M:%S\n==========\n')
                cpustr = format_cpu(psutil.cpu_percent(interval=.1, percpu=True))
                memstr = format_mem(psutil.virtual_memory())
                diskstr = format_disk(psutil.disk_usage('/'))
                
                status = (timestr + cpustr + memstr + diskstr + '\n')
                
                self.status.state = status
                self.status.push_threadsafe()
                
                elapsed = (datetime.datetime.now() - timestamp).total_seconds()
                print('Logged in {:.3f} seconds:\n{}'.format(elapsed, status))
                # Make sure we clamp this to non-negative values, in case the
                # update took longer than the current interval.
                time.sleep(max(self.interval - elapsed, 0))
            
        def pair_handler(self, ghid, origin, api_id):
            ''' Pair handlers ignore the object itself, instead setting up
            the origin as the paired_fingerprint (unless one already exists,
            in which case it is ignored) and sharing the status object with
            them.
            
            This also doubles as a way to re-pair the same fingerprint, in
            the event that they have gone offline for a long time and are no
            longer receiving updates.
            '''
            # The initial pairing (pair/trust on first connect)
            if self.paired_fingerprint is None:
                self.paired_fingerprint = origin
            
            # Subsequent pairing requests from anyone else are ignored
            elif self.paired_fingerprint != origin:
                return
                
            # Now we want to share the status reporter, if we have one, with the
            # origin
            if self.status is not None:
                self.status.share_threadsafe(origin)
                
        def interval_handler(self, ghid, origin, api_id):
            ''' Interval handlers change our recording interval.
            '''
            # Ignore requests that don't match our pairing.
            # This will also catch un-paired requests.
            if origin != self.paired_fingerprint:
                return
            
            # If the address matches our pairing, use it to change our interval.
            else:
                # We don't need to create an update callback here, because any
                # upstream modifications will automatically be passed to the
                # object. This is true of all hypergolix objects, but by using a
                # proxy, it mimics the behavior of the int itself.
                interval_proxy = self.hgxlink.get_threadsafe(
                    cls = hgx.JsonProxy,
                    ghid = ghid
                )
                self._interval = interval_proxy
                
        @property
        def interval(self):
            ''' This provides some consumer-side protection against
            malicious interval proxies.
            '''
            try:
                return float(max(self._interval, self.minimum_interval))
            
            except (ValueError, TypeError):
                return self.minimum_interval


    class Monitor:
        ''' Remote monitoring demo app receiver.
        '''
        
        def __init__(self, telemeter_fingerprint):
            self.hgxlink = hgx.HGXLink()
            self.telemeter_fingerprint = telemeter_fingerprint
            
            # These are the actual Hypergolix business parts
            self.status = None
            self.pair = None
            self.interval = None
            
        def app_init(self):
            ''' Set up the application.
            '''
            # Because we're using a native coroutine for this share handler, it
            # needs no wrapping.
            self.hgxlink.register_share_handler_threadsafe(STATUS_API,
                                                           self.status_handler)
            
            # Wait until after registering the share handler to avoid a race
            # condition with the Telemeter
            self.pair = self.hgxlink.new_threadsafe(
                cls = hgx.JsonObj,
                state = 'Hello world!',
                api_id = PAIR_API
            )
            self.pair.share_threadsafe(self.telemeter_fingerprint)
            
        def app_run(self):
            ''' For now, just busy-wait.
            '''
            while True:
                time.sleep(1)
            
        async def status_handler(self, ghid, origin, api_id):
            ''' We sent the pairing, and the Telemeter shared its status obj
            with us in return. Get it, store it locally, and register a
            callback to run every time the object is updated.
            '''
            status = await self.hgxlink.get(
                cls = hgx.JsonObj,
                ghid = ghid
            )
            # This registers the update callback. It will be run in the hgxlink
            # event loop, so if it were blocking/threaded, we would need to wrap
            # it like this: self.hgxlink.wrap_threadsafe(self.update_handler)
            status.callback = self.update_handler
            # We're really only doing this to prevent garbage collection
            self.status = status
            
        async def update_handler(self, obj):
            ''' A very simple, **asynchronous** handler for status updates.
            This will be called every time the Telemeter changes their
            status.
            '''
            print(obj.state)
            
        def set_interval(self, interval):
            ''' Set the recording interval remotely.
            '''
            # This is some supply-side protection of the interval.
            interval = float(interval)
            
            if self.interval is None:
                self.interval = self.hgxlink.new_threadsafe(
                    cls = hgx.JsonProxy,
                    state = interval,
                    api_id = INTERVAL_API
                )
                self.interval.hgx_share_threadsafe(self.telemeter_fingerprint)
            else:
                # We can't directly reassign the proxy here, because it would just
                # overwrite the self.interval name with the interval float from
                # above. Instead, we need to assign to the state.
                self.interval.hgx_state = interval
                self.interval.hgx_push_threadsafe()


    if __name__ == "__main__":
        argparser = argparse.ArgumentParser(
            description = 'A simple remote telemetry app.'
        )
        argparser.add_argument(
            '--telereader',
            action = 'store',
            default = None,
            help = 'Pass a Telemeter fingerprint to run as a reader.'
        )
        argparser.add_argument(
            '--interval',
            action = 'store',
            default = None,
            type = float,
            help = 'Set the Telemeter recording interval from the Telereader. ' +
                   'Ignored by a Telemeter.'
        )
        args = argparser.parse_args()
        
        if args.telereader is not None:
            telemeter_fingerprint = hgx.Ghid.from_str(args.telereader)
            app = Monitor(telemeter_fingerprint)
            
            try:
                app.app_init()
                
                if args.interval is not None:
                    app.set_interval(args.interval)
                
                app.app_run()
                
            finally:
                app.hgxlink.stop_threadsafe()
            
        else:
            app = Telemeter(interval=5)
                
            try:
                app.app_init()
                app.app_run()
                
            finally:
                app.hgxlink.stop_threadsafe()

-------------------------------------------------------------------------------
One last thing: daemonizing
-------------------------------------------------------------------------------

Our simple server monitoring app works pretty well, but there's still one
problem left: the telemeter cannot run on its own. If we, for example, run it
in the background using ``python telemeter.py &``, it will shut down as soon
as our shell exits (in other words, if we're working via SSH, as soon as our
session disconnects). To keep it running, we need to properly daemonize the
script.

A full discussion of daemonization is out-of-scope for Hypergolix, but if you
want to learn more, check out the
`Daemoniker documentation <http://daemoniker.readthedocs.io/en/latest/>`_.
Regardless, the following changes will keep our script running in the
background until we explicitly stop it:

.. sidebar::

    Don't forget to ``pip install daemoniker`` into the environment!
    
.. code-block:: python

    import argparse
    import time
    import datetime
    import psutil
    import collections
    import daemoniker
    import hgx


    # These are app-specific (here, totally random) API schema identifiers
    STATUS_API = hgx.utils.ApiID(
        b'\x02\x0b\x16\x19\x00\x19\x10\x18\x08\x12\x03' +
        b'\x11\x07\x07\r\x0c\n\x14\x04\x13\x07\x04\x06' +
        b'\x13\x01\x0c\x04\x00\x0b\x03\x01\x12\x05\x0f' +
        b'\x01\x0c\x05\x11\x03\x01\x0e\x13\x16\x13\x11' +
        b'\x10\x13\t\x06\x10\x00\x14\x0c\x15\x0b\x07' +
        b'\x0c\x0c\x04\x07\x0b\x0f\x18\x03'
    )
    PAIR_API = hgx.utils.ApiID(
        b'\x17\n\x12\x17\x03\x0f\x14\x11\x07\x10\x05\x04' +
        b'\x14\x18\x11\x11\x12\x02\x17\x12\x15\x0e\x04' +
        b'\x0f\x11\x19\x07\x19\n\r\x03\x06\x12\x04\x17' +
        b'\x11\x14\x07\t\x08\x13\x19\x04\n\x0f\x15\x12' +
        b'\x14\x07\x19\x16\x13\x18\x0b\x18\x0e\x12\x15\n' +
        b'\n\x16\x0f\x08\x14'
    )
    INTERVAL_API = hgx.utils.ApiID(
        b'\n\x10\x04\x00\x13\x11\x0b\x11\x06\x02\x19\x00' +
        b'\x11\x12\x10\x10\n\x14\x19\x15\x11\x18\x0f\x0f' +
        b'\x01\r\x0c\x15\x16\x04\x0f\x18\x19\x13\x14\x11' +
        b'\x10\x01\x19\x19\x15\x0b\t\x0e\x15\r\x16\x15' +
        b'\x0e\n\x19\x0b\x14\r\n\x04\x0c\x06\x03\x13\x01' +
        b'\x01\x12\x05'
    )


    def humanize_bibytes(n, prefixes=collections.OrderedDict((
                        (0, 'B'),
                        (1024, 'KiB'),
                        (1048576, 'MiB'),
                        (1073741824, 'GiB'),
                        (1099511627776, 'TiB'),
                        (1125899906842624, 'PiB'),
                        (1152921504606846976, 'EiB'),
                        (1180591620717411303424, 'ZiB'),
                        (1208925819614629174706176, 'YiB')))):
        ''' Convert big numbers into easily-human-readable ones.
        '''
        for value, prefix in reversed(prefixes.items()):
            if n >= value:
                return '{:.2f} {}'.format(float(n) / value, prefix)
                
                
    def format_cpu(cpu_list):
        cpustr = 'CPU:\n----------\n'
        for cpu in cpu_list:
            cpustr += '  ' + str(cpu) + '%\n'
        return cpustr
        
        
    def format_mem(mem_tup):
        memstr = 'MEM:\n----------\n'
        memstr += '  Avail: ' + humanize_bibytes(mem_tup.available) + '\n'
        memstr += '  Total: ' + humanize_bibytes(mem_tup.total) + '\n'
        memstr += '  Used:  ' + str(mem_tup.percent) + '%\n'
        return memstr
        
        
    def format_disk(disk_tup):
        diskstr = 'DISK:\n----------\n'
        diskstr += '  Avail: ' + humanize_bibytes(disk_tup.free) + '\n'
        diskstr += '  Total: ' + humanize_bibytes(disk_tup.total) + '\n'
        diskstr += '  Used:  ' + str(disk_tup.percent) + '%\n'
        return diskstr


    class Telemeter:
        ''' Remote monitoring demo app sender.
        '''
        
        def __init__(self, interval, minimum_interval=1):
            self.hgxlink = hgx.HGXLink()
            self._interval = interval
            self.minimum_interval = minimum_interval
            
            # These are the actual Hypergolix business parts
            self.status = None
            self.paired_fingerprint = None
            
            self.running = True
            
        def app_init(self):
            ''' Set up the application.
            '''
            # print('My fingerprint is: ' + self.hgxlink.whoami.as_str())
            self.status = self.hgxlink.new_threadsafe(
                cls = hgx.JsonObj,
                state = 'Hello world!',
                api_id = STATUS_API
            )
            
            # Share handlers are called from within the HGXLink event loop, so they
            # must be wrapped before use
            pair_handler = self.hgxlink.wrap_threadsafe(self.pair_handler)
            self.hgxlink.register_share_handler_threadsafe(PAIR_API, pair_handler)
            # And set up a handler to change our interval
            interval_handler = self.hgxlink.wrap_threadsafe(self.interval_handler)
            self.hgxlink.register_share_handler_threadsafe(INTERVAL_API,
                                                           interval_handler)
            
        def app_run(self):
            ''' Do the main application loop.
            '''
            while self.running:
                timestamp = datetime.datetime.now()
                timestr = timestamp.strftime('%Y.%m.%d @ %H:%M:%S\n==========\n')
                cpustr = format_cpu(psutil.cpu_percent(interval=.1, percpu=True))
                memstr = format_mem(psutil.virtual_memory())
                diskstr = format_disk(psutil.disk_usage('/'))
                
                status = (timestr + cpustr + memstr + diskstr + '\n')
                
                self.status.state = status
                self.status.push_threadsafe()
                
                elapsed = (datetime.datetime.now() - timestamp).total_seconds()
                # print('Logged in {:.3f} seconds:\n{}'.format(elapsed, status))
                # Make sure we clamp this to non-negative values, in case the
                # update took longer than the current interval.
                time.sleep(max(self.interval - elapsed, 0))
                
        def signal_handler(self, signum):
            self.running = False
            self.hgxlink.stop_threadsafe()
            
        def pair_handler(self, ghid, origin, api_id):
            ''' Pair handlers ignore the object itself, instead setting up
            the origin as the paired_fingerprint (unless one already exists,
            in which case it is ignored) and sharing the status object with
            them.
            
            This also doubles as a way to re-pair the same fingerprint, in
            the event that they have gone offline for a long time and are no
            longer receiving updates.
            '''
            # The initial pairing (pair/trust on first connect)
            if self.paired_fingerprint is None:
                self.paired_fingerprint = origin
            
            # Subsequent pairing requests from anyone else are ignored
            elif self.paired_fingerprint != origin:
                return
                
            # Now we want to share the status reporter, if we have one, with the
            # origin
            if self.status is not None:
                self.status.share_threadsafe(origin)
                
        def interval_handler(self, ghid, origin, api_id):
            ''' Interval handlers change our recording interval.
            '''
            # Ignore requests that don't match our pairing.
            # This will also catch un-paired requests.
            if origin != self.paired_fingerprint:
                return
            
            # If the address matches our pairing, use it to change our interval.
            else:
                # We don't need to create an update callback here, because any
                # upstream modifications will automatically be passed to the
                # object. This is true of all hypergolix objects, but by using a
                # proxy, it mimics the behavior of the int itself.
                interval_proxy = self.hgxlink.get_threadsafe(
                    cls = hgx.JsonProxy,
                    ghid = ghid
                )
                self._interval = interval_proxy
                
        @property
        def interval(self):
            ''' This provides some consumer-side protection against
            malicious interval proxies.
            '''
            try:
                return float(max(self._interval, self.minimum_interval))
            
            except (ValueError, TypeError):
                return self.minimum_interval


    class Monitor:
        ''' Remote monitoring demo app receiver.
        '''
        
        def __init__(self, telemeter_fingerprint):
            self.hgxlink = hgx.HGXLink()
            self.telemeter_fingerprint = telemeter_fingerprint
            
            # These are the actual Hypergolix business parts
            self.status = None
            self.pair = None
            self.interval = None
            
        def app_init(self):
            ''' Set up the application.
            '''
            # Because we're using a native coroutine for this share handler, it
            # needs no wrapping.
            self.hgxlink.register_share_handler_threadsafe(STATUS_API,
                                                           self.status_handler)
            
            # Wait until after registering the share handler to avoid a race
            # condition with the Telemeter
            self.pair = self.hgxlink.new_threadsafe(
                cls = hgx.JsonObj,
                state = 'Hello world!',
                api_id = PAIR_API
            )
            self.pair.share_threadsafe(self.telemeter_fingerprint)
            
        def app_run(self):
            ''' For now, just busy-wait.
            '''
            while True:
                time.sleep(1)
            
        async def status_handler(self, ghid, origin, api_id):
            ''' We sent the pairing, and the Telemeter shared its status obj
            with us in return. Get it, store it locally, and register a
            callback to run every time the object is updated.
            '''
            print('Incoming status: ' + ghid.as_str())
            status = await self.hgxlink.get(
                cls = hgx.JsonObj,
                ghid = ghid
            )
            # This registers the update callback. It will be run in the hgxlink
            # event loop, so if it were blocking/threaded, we would need to wrap
            # it like this: self.hgxlink.wrap_threadsafe(self.update_handler)
            status.callback = self.update_handler
            # We're really only doing this to prevent garbage collection
            self.status = status
            
        async def update_handler(self, obj):
            ''' A very simple, **asynchronous** handler for status updates.
            This will be called every time the Telemeter changes their
            status.
            '''
            print(obj.state)
            
        def set_interval(self, interval):
            ''' Set the recording interval remotely.
            '''
            # This is some supply-side protection of the interval.
            interval = float(interval)
            
            if self.interval is None:
                self.interval = self.hgxlink.new_threadsafe(
                    cls = hgx.JsonProxy,
                    state = interval,
                    api_id = INTERVAL_API
                )
                self.interval.hgx_share_threadsafe(self.telemeter_fingerprint)
            else:
                # We can't directly reassign the proxy here, because it would just
                # overwrite the self.interval name with the interval float from
                # above. Instead, we need to assign to the state.
                self.interval.hgx_state = interval
                self.interval.hgx_push_threadsafe()


    if __name__ == "__main__":
        argparser = argparse.ArgumentParser(
            description = 'A simple remote telemetry app.'
        )
        argparser.add_argument(
            '--telereader',
            action = 'store',
            default = None,
            help = 'Pass a Telemeter fingerprint to run as a reader.'
        )
        argparser.add_argument(
            '--interval',
            action = 'store',
            default = None,
            type = float,
            help = 'Set the Telemeter recording interval from the Telereader. ' +
                   'Ignored by a Telemeter.'
        )
        argparser.add_argument(
            '--pidfile',
            action = 'store',
            default = 'telemeter.pid',
            type = str,
            help = 'Set the name for the PID file for the Telemeter daemon.'
        )
        argparser.add_argument(
            '--stop',
            action = 'store_true',
            help = 'Stop an existing Telemeter daemon.'
        )
        args = argparser.parse_args()
        
        # This is the READER
        if args.telereader is not None:
            telemeter_fingerprint = hgx.Ghid.from_str(args.telereader)
            app = Monitor(telemeter_fingerprint)
            
            try:
                app.app_init()
                
                if args.interval is not None:
                    app.set_interval(args.interval)
                
                app.app_run()
                
            finally:
                app.hgxlink.stop_threadsafe()
        
        # This is the SENDER, but we're stopping it.
        elif args.stop:
            daemoniker.send(args.pidfile, daemoniker.SIGTERM)
            
        # This is the SENDER, and we're starting it.
        else:
            # We need to actually daemonize the app so that it persists without
            # an SSH connection
            with daemoniker.Daemonizer() as (is_setup, daemonizer):
                is_parent, pidfile = daemonizer(
                    args.pidfile,
                    args.pidfile,
                    strip_cmd_args = False
                )
                
                # Parent exits here
            
            # Just the child from here
            app = Telemeter(interval=5)
                
            try:
                sighandler = daemoniker.SignalHandler1(
                    pidfile,
                    sigint = app.signal_handler,
                    sigterm = app.signal_handler,
                    sigabrt = app.signal_handler
                )
                sighandler.start()
                
                app.app_init()
                app.app_run()
                
            finally:
                app.hgxlink.stop_threadsafe()
