************
DIAS Logging
************

The logging system used here can be found on |logging-system|.

The instructions on how to start the logging daemon can be found in the :ref:`Launching DIAS` section.

There are 3 different Loggers, that can be used:


* :ref:`RawLog`
* :ref:`EventLog`
* :ref:`MemLog`

Before any of those loggers can be used, the persistence client needs to be set up:

.. code-block:: Java

  import pgpersist.PersistenceClient;
  /*...*/
  persistenceClient = new PersistenceClient(zmqContext, daemonConnectString, persistenceClientOutputQueueSize);


RawLog
------

* RawLog is designed as a drop-copy replacement for System.out.printf
* The function takes 2 arguments

  * The log level, a number between 1 and 3 representing respectively an information, a warning or an error message
  * Any string to be persisted
* The RawLog automatically records the id of the calling thread allowing insights into potential issues regarding thread synchronisation

Initialisation
##############

To create the RawLog, the persistenceClient needs to be set, as well as the logging level threshold, the peerID, and the networkID:

.. code-block:: Java

  RawLog.setPeristenceClient(persistenceClient);
  RawLog.setErrorLevelThreshold(rawLogLevel); // log all (1); only log warnings (2); errors (3)
  RawLog.setPeer(newPeer);
  RawLog.setDIASNetworkId(diasNetworkId);

Usage
#####

To log something, one just has to insert the following line:

.. code-block:: Java

  RawLog.print(1,"Text to be logged");

The '1' specifies the error level, there is an internal level threshold, which can be set to a specific level.
**Every level smaller to the threshold will not get logged.**

SQL Table
#########

The underlying sql table looks like this:

.. list-table::
   :header-rows: 1

   * - SQL field definition
     - Description
   * - seq_id SERIAL NOT NULL
     - The entry sequence id, created automatically
   * - dt TIMESTAMP NOT NULL
     - Timestampt of logged entry, measured automatically by the RawLog class
   * - network INTEGER NOT NULL DEFAULT 0
     - The network id for the logged entry, 0 if not defined
   * - peer INTEGER NOT NULL
     - The peer number
   * - epoch BIGINT NOT NULL
     - The epoch is System.currentTimeMillis()/1000
   * - thread_id INTEGER NOT NULL
     - The id of the thread triggering the log, measured automatically
   * - error_level INTEGER NOT NULL
     - The error level of the logged event, 1 is normal, 2 is warning and 3 is error
   * - txt TEXT
     - The logged text

EventLog
--------

* EventLog is designed to understand code execution sequence within a peer and across multiple peers
* It is different than RawLog in that it is structured, allowing for side-by-side comparison of events within and across peers
* Furthermore, the EventLog automatically records the id of the calling thread allowing insights into potential issues regarding thread synchronisation

Initialisation
##############

The initialisation is similar to the :ref:`RawLog`:

.. code-block:: Java

  EventLog.setPeristenceClient(persistenceClient);
  EventLog.setPeer(newPeer);
  EventLog.setDIASNetworkId(diasNetworkId);

Usage
#####

The logging function takes 4 arguments:

  * The name of the calling class, e.g. DIAS
  * The name of the calling function, e.g. start
  * A key, e.g. "alreadyStarted"
  * A value (text), e.g. "True"

.. code-block:: Java

  EventLog.logEvent("DIAS", "start", "alreadyStarted", Boolean.toString(alreadyStarted));

SQL Table
#########

.. list-table::
   :header-rows: 1

   * - SQL field definition
     - Description
   * - seq_id SERIAL NOT NULL
     - The entry sequence id, created automatically
   * - dt TIMESTAMP NOT NULL
     - Timestampt of logged entry, measured automatically by the RawLog class
   * - current_time_millis BIGINT NOT NULL
     - The current time in milliseconds
   * - network INTEGER NOT NULL DEFAULT 0
     - The network id for the logged entry, 0 if not defined
   * - peer INTEGER NOT NULL
     - The peer number
   * - epoch BIGINT NOT NULL
     - The epoch is System.currentTimeMillis()/1000
   * - peer_event_counter BIGINT NOT NULL
     - An internal counter of the events inside the peer
   * - thread_id INTEGER NOT NULL
     - The id of the thread triggering the log, measured automatically
   * - classname TEXT NOT NULL
     - The name of the caller class
   * - func TEXT NOT NULL
     - The name of the caller function
   * - key TEXT
     - A key string for categorinsing the log
   * - value TEXT
     - The message to be logged

MemLog
------

* MemLog is designed to periodically measure total memory footprint of any Java object in memory
* It's great for detecting memory leaks that appear over time, e.g over several weeks of operation
* As explained below, MemLog uses the JAMM instrumentation agent, that traverses all elements and member variables of complex Java objects such as classes and containers

Initialisation
##############

* The MemLog class is initialised similarly to the RawLog and EventLog classes above, with some notable differences
* First, a Java instrumentation class, that will take the actual memory measurements, is required for the MemLog and must be passed as an argument to Memlog. More information about Java instrumentation can be found at TODO

  * The particular instrumentation class used is JAMM (see references below). JAMM is a powerful Java instrumentation class that mesures the size of objects in memory. For containers, JAMM actually traverses the containers and measures the size of each element, further traversing each element if it is a container. This recursive approach allows proper estimation of the memory footprint of an object.

* Second, the period for taking snapshots

  * Memlog will periodically compute the memory footprint of each object that was added with add_object(), at the period specificed in startTimer()

.. code-block:: Java

  // initialise MemLog
  final MemoryMeter instrument = new MemoryMeter();
  if(!MemoryMeter.hasInstrumentation())
    System.out.printf( "MemLog: No instrumentation\n");
  else {
    MemLog.setPersistenceClient(persistenceClient);
    MemLog.setMeasurementInstrument(instrument);
    MemLog.setPeer(newPeer);
    MemLog.setDIASNetworkId(diasNetworkId);
    MemLog.startTimer(60);

    System.out.printf( "MemLog setup\n" );
    MemLog.add_object("Peer", "Peer", newPeer);
  }

**Note:** the timer period is specified in *seconds* and **not** in milliseconds.


Usage
#####

* Objects where memory monitoring is required need to be added with a call to add_object, that takes 3 arguments

  * Arguments one and two are simple tags that will allow you

    * In the example below, we are using the class in which the object belongs and the name of the instantiated object

  * The third argument is a reference to the Java object to be monitored

* Once an object has been added, as per the example below, a sample of the total memory footprint of the object will be taken every 60 seconds and persisted to the memlog table

.. code-block:: Java

  MemLog.add_object("DIASlight", "dumper", this.dumper);

**Note:** dumper is just a random object to be measured.


SQL Table
#########

.. list-table::
   :header-rows: 1

   * - SQL field definition
     - Description
   * - seq_id SERIAL NOT NULL
     - The entry sequence id, created automatically
   * - dt TIMESTAMP NOT NULL
     - Timestampt of logged entry, measured automatically by the RawLog class
   * - network INTEGER NOT NULL DEFAULT 0
     - The network id for the logged entry, 0 if not defined
   * - peer INTEGER NOT NULL
     - The peer number
   * - epoch BIGINT NOT NULL
     - The epoch is System.currentTimeMillis()/1000
   * - object_group_name TEXT NOT NULL
     - A groupt name to categorise the objects in groups
   * - object_name TEXT NOT NULL
     - The name of the object
   * - object_size_mb FLOAT
     - Can be null if the object is null



.. |logging-system| raw:: html

   <a href="https://github.com/epournaras/Livelog" target="_blank">Github</a>
