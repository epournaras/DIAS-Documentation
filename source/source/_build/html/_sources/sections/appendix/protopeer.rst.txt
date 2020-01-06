.. _protopeer-chapter:

Protopeer
*********

Introduction
------------

In this section we which review the improvements that were made to the Protopeer Framework. The section is organised in two sub-sections, the first covering memory leaks and the second networking optimisations.

The Protopeer Framework was designed in 2009 by two researches from the Swiss Institute of Technology in Lausanne (Galuba 2009) with the intention of providing a robust platform for carrying out small to medium scale peer-to-peer experiments. One important characteristic of the Protopeer Framework is the ability to carry out the experiments in either simulated mode or real-time mode, with minimal (if any) change to the code. In simulated mode, all the peers reside in a single process running inside a single Java Virtual Machine instance on a single machine. This mode facilitates development and debugging. In real-time mode, each peer is a separate process running inside its own Java Virtual Machine instance which can be located anywhere on the Internet.

At the time of the initial release, the Protopeer Framework was the only platform with that capability.

In 2016 we started working on the real-time implementation of DIAS. Back then, the Protopeer Framework still had many low-level technical issues with its real-time mode such as memory leaks, static port allocation, etc All of these issues have been solved to allow stable real-time DIAS simulations and real-time experiments.

The issues belonged to two broad categories, which will be covered in subsections below.
1. Memory Leaks
2. Networking Optimisations

Memory Leaks
************

Back in 2016, it was not possible to run experiments using the Protopeer Framework in real-time mode for longer than a day. This was due to extensive memory leaks in various parts of the Protopeer library.

The exact cause and code location of the memory leaks was determined using Java's hprof profiling tool. A sample of the code required to launch the profiler is provided below. Given that the classes in the Protopeer Framework can be derived from parent clases many layers deep, it was necessary to performing the profiling with up to 20 function calls (parameter tracedepth).

.. code:: bash

	# ... this is your bash script to launch a Protopeer real-time experiment
	profileoutput=profile/dump.out
	tracedepth=20

	profilearg="-Xrunhprof:interval=60000,depth=$tracedepth ,heap=sites,format=a,file=$profileoutput"

	java -Xmx$javamem $profilearg -cp "$classpath" ...

Figure {x}. Launch commands for hprof profiling

The result of the profiling showed three areas that required attention.

Apache Mina
-----------

The first memory leak was detected Apache Mina, the networking library used for inter-peer TCP communication. Apache MINA is a network application framework which helps users develop high performance and high scalability network applications easily (src: https://mina.apache.org/). We favored the use of migrating to the ZeroMQ messaging framework, for the following reasons:
1. ZeroMQ can be easily integrated into other programming languages and other devices, allowing for multi-peer networking with peers written in different programming languages to communicate seamlessly
2. The ZeroMQ libraries are far simpler to use
3. The size of the library is smaller
4. Our development team had good prior experience with ZeroMQ

We implemented JeroMQ, a native zeroMQ port for java (src: https://github.com/zeromq/jeromq), and tested versions 2, 3 and 4.

Starting a server in JeroMQ requires only a few lines of code.

.. code:: java
	
	final ZMQ.Context   zmqContext = ZMQ.context(1);

	final ZMQ.Socket    zmqSocket= zmqContext.socket(ZMQ.REP);

	final int zmqListenPort = zmqSocket.bindToRandomPort(ConnectString);

Figure {x}. Starting a server with JeroMQ


ByteBuffer.allocate()
---------------------

Another memory leak was in repeated calls to ByteBuffer.allocate() made throughout the codebase. In the Protopeer Framework, byte buffers are used when serialising objects, e.g messages to be sent over the network. The problem is that ByteBuffer.allocate() allocates an array of bytes that bypasses Java GC, leading to memory leaks if the allocation is done repeatedly during the programs lifetime. (More information can be found here: http://www.evanjones.ca/java-bytebuffer-leak.html).

We modified the Protopeer Framework so that a singleByteBuffer is only allocated once per class instance, at class construction.

.. code:: java

	final int         strlen = msg.length();
       
	ByteBuffer bb = ByteBuffer.allocate(strlen).order(ByteOrder.nativeOrder());
	bb.put(msg.getBytes(ZMQ.CHARSET));
	bb.flip();
       
	final int ret = ZMQSocket.sendByteBuffer(bb, 0);

Figure {x}. Sample use of ByteBuffer.allocate() that will lead to a memory leak if called repeatedly during a programs lifetime


MeasurementLog
--------------

The Protopeer Framework provides support for recording an arbitrary number of values during an experiment, greatly facilitating post-experiment analyses. The measurement log also supports serialising the measurement log to a file, through a class callsed MeasurementFileDumper.  A memory leak existed because of ObjectOutputStream, a Java utility class that serialises Java objects to a stream (in our case, we are serialising to a file). ObjectOutputStream maintains a reference to each serialised object when writeObject is called, and these references are not released by default. This leads to object references that cannot be garbage-collected. As shown in the example below, it is required to call the function reset() to release the reference.

.. code:: java

	public class MeasurementFileDumper implements MeasurementLoggerListener 
	{
	    private ObjectOutputStream measurementsOut;

		public void measurementEpochEnded(MeasurementLog log, int epochNumber) 
    		{
		        ...
			measurementsOut.writeObject(subLog);

			// reset() needs to be added to release the object inside the serialisation, else it will cause a memory leak
			measurementsOut.reset();        
			measurementsOut.flush();

		}
	}

Figure {x}. Memory leak fixed by calling reset() method of the ObjectOutputStream object

Networking Optimisations
************************

**1. Off-thread message sending**

The Protopeer Framework is a multi-threaded library. Threads are used to handle incoming messages (also known as the passive state) as well as repeated events using timers, of which the active state is such an event. An issue that we identified is that multiple threads may need to send a network message to another peer(s) at the same time, leading to contention and inducing delays in the higher-level applcations of the peer such as DIAS or the Peer Sampling Service. This problem is acute for a real-time system because the worker threads cannot send a network message if another worker thread is doing the same, leading to idle waiting times in the worker threads.

We alleviated this problem by handling the actual sending of messages in a background thread. In the new design, the worker threads only add messages to a FIFO queue (implemented as a circular buffer), and can then return to their work. In the background, a dedicated thread is continuously working to send all messages still in the queue. This design allows the worker threads to effectively delegate the lower level operations related to the sending of a message.

It is important to note that the message to be sent is serialised by the worker thread before being added to the queue. Delegating the serialisation of the message to the background thread may appear more optimal - it is actually error prone since it leads to synchronization issue. For example, a LEAVE message contains a reference to a peer's disseminator. The serialisation needs to occur immediately, not at a later stage when then the state of the disseminator may have changed due to subsequent interactions with other peers.

.. code:: java

	zmqOutboundMsgQueueProcessor.add_message_to_queue(new SerialisedMessage(message) );

Figure {x}. Serialisation of messages by the worker thread and adding to the outbound message queue (ZMQNetworkInterface:155)

**2. Dynamic Port Allocation**

Previously the port number of the socket through which the Peer interacted with other peers was determined by the startup script and passed as an argument to the java executable. This caused problems when scaling to hundreds of peers on a single host as this conflicted where other applications on the host that used network sockets. Thus the design was changed to allow dynamic allocation of ports using ZeroMQ::bind function, which automatically searches in the host's dynamic port range for the next available port. The dynamic port range is 49152 to 65535. The Protopeer Framework fully supports dynamic port allocation since each peer communicates it's own IP address to the DIAS Boostrap Server on startup, so the operation is not affected by ports being statically or dynamically defined.

.. code:: java

	final ZMQ.Context   zmqContext = ZMQ.context(1);

	final  ZMQ.Socket    zmqSocket= zmqContext.socket(ZMQ.REP);

	final int zmqListenPort = zmqSocket.bindToRandomPort(ConnectString);

Figure {x}. Dynamic port assignment within ZeroMQ (ZeroMQReqRepListener:74)

**3. Connection pool**

In peer-to-peer networks such as DIAS, peers communicate with many other peers. The interactions can be brief to long running, depending on the size of the network and it's topology. As a result a long-running peer may communicate with many other peers during operation.

In the initial version of the Protopeer Framework, each time a peer sent a message to another peer, a new connection with the target peer was created, the message was sent, and the connection was immediately dropped. This is admittedly an inefficient process especially as there can be many messages exchange per second.

We improved the framework by allowing a connection pool of 200 peers. That is, a FIFO hashmap of open connections is maintained, so that existing open connections with peers can be re-used where possible. Should a peer runs out of available connections, an older connection is closed and a new connection is created. This process is managed in the background and entirely transparent to the high-level peerlets such as DIAS and Peer Sampling Service.

.. code:: java

	public LinkedHashMap<String,ZMQ.Socket>    pushSockets = new LinkedHashMap<String,ZMQ.Socket>();

	// close a socket if too many open
	// maxNumOpenSockets presently set to 200
	if( numOpenSockets >= maxNumOpenSockets )
	{
	    // find a socket to close
	    if( this.debug_level2) System.out.printf( "need to close a socket\n" );
                    
	    // get the first one in the map 
	    // TODO: ideally should close the least used socket (time weighted)
	    String addressToCloseString = pushSockets.entrySet().iterator().next().getKey();
                
	    if( this.debug_level2) System.out.printf( "addressToCloseString : %s\n", addressToCloseString );
                
	    // close socket and remove from hash map
	    closeConnectionStr(addressToCloseString);
	}

Figure {x}. Implementation of the connection pool in the Protopeer Framework. ZMQNetworkInterface:222


References
----------

Galuba W., Aberer K. (2009) ProtoPeer-A P2P Toolkit Bridging the Gap Between Simulation and Live Deployment