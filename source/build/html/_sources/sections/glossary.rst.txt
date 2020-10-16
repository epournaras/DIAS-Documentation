Glossary
********

.. list-table::
  :header-rows: 1

  * - Expression
    - Brief explanation
  * - Active state
    - Every DIAS node has an active state, it is basically an acttive thread running on a timer, repeatedly doing specific tasks.
  * - Aggregator
    - A peerlet in a peer, the aggregator is responsible for making its local aggregations on the values of the whole network.
  * - Applet
    - Little programs, which execute a specific tast.
  * - Bloom Filter
    - A bloom filter is a way to store much information in little room, with the tradeoff for false positives. |BloomFilter|
  * - DIAS
    - The name of the hereby described project, see :ref:`DIAS - Dynamic Intelligent Aggregation Service`
  * - DIAS Bootstrap server
    - The server, which initially interconnects the first peers in the DIAS network, see :ref:`Bootstrap Process`
  * - DIAS Gateway Server
    - The server connecting newly created peer to the already running network, see :ref:`DIAS Gateway`
  * - DIAS-Logging-System
    - The system, which allows the distributed system to log averything to a single database with relatively little contention, see :ref:`DIAS Logging`
  * - DIAS Network Id
    - The Network Id (oe sensor ID) is used to distinguish between different peers to be able to address a specific peer, see more here: :ref:`The usage of sensorIDs`,
  * - DIAS Node
    - One aggregating node inside the DIAS network
  * - DIAS Peer
    - One participant in the DIAS network
  * - Disseminator
    - A peerlet in a peer, it is responsible mor the communication between the different peers, as well as keeping track what information was disseminated to what peer and which information is still valid.
  * - EventLog
    - This logger logs the data more ordered, giving the option to track which object and method invoked the logging, as well as being able to provide a specific key to filter out specific logs.
  * - GDELT
    - GDELT is the largest publicly available collection of near real-time news feed in the World, see :ref:`GDELT`
  * - JVM
    - The Java Virtual Machine is a vm, in which the java program is running, it encapsulates the program and contains all data to run it.
  * - MemLog
    - This log is automated and tracks the memory usage of the given objects.
  * - Memory Leak
    - Incorrectly managed memory leads to allocated memory, which after not needing it anymore does not get freed again and remains in the program for the rest of its lifecycle. This results in trash laying in memory, filling up the memory usage.
  * - Mock Device
    - A program simulating the input of real-world users for different experiments, see :ref:`Mock devices`
  * - Passive state
    - Every DIAS node has a passive state, it gets triggered upon receiving a message, handling it corresponding to the defined actions.
  * - Peer Sampling Service
    - The Peer Sampling Service takes a sample of all nodes in the network, it may be every peer, if not many peers exist, or a subset, if the peer set is large.
  * - Peerlet
    - A submodule in a Protopeer peer
  * - Possible states
    - The user needs to send a node different options, from which he later chooses a single one as a selected state. see :ref:`Changing of possible states`
  * - Postgres
    - A popular implementation of the SQL database
  * - Protopeer Framework
    - A platform written in Java for carrying out small to medium scale peer-to-peer experiments, see :ref:`Protopeer`
  * - Servlet
    - Java classes, which run on a server, serving user requests.
  * - RawLog
    - A single String message is logged.
  * - Selected state
    - The selected state is a selected element out of the possible states, it is considered as the current active value sent from a device. see :ref:`Changing of selected states`
  * - UNIX Screen
    - A UNIX command line tool, read |UNIXScreen|
  * - ZeroMQ
    - ZeroMQ is a low level, high efficiency communication library, based on TCP. It is available for many platforms. 

.. |UNIXScreen| raw:: html

  <a href="https://kb.iu.edu/d/acuy" target="_blank">more</a>

.. |BloomFilter| raw:: html

  <a href="https://de.wikipedia.org/wiki/Bloomfilter" target="_blank">Read more</a>

.. |ZeroMQ| raw:: html

  <a href="https://zeromq.org" target="_blank">Read more</a>
