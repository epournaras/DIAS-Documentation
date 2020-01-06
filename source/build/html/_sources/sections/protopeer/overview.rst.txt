.. _label_protopeer_overview:

Protopeer Overview
******************

Protopeer is a Java framework allowing rapid prototying of fully decentralised peer-to-peer networks. It was designed by Wojciech Galuba and Karl Aberer in 2008. The original research paper can be found here (http://lsirpeople.epfl.ch/galuba/papers/p2p08.pdf).

It has the advantage of allowing the same code to be developed for simulation and real-time deployments. In simulation mode, large networks of several thousand peers can be tested within a single JVM, but the peers don't work with real-time data.

On the other hand, in real-time mode, each peer is launched separately and runs in inside it's own JVM. The advantage is that the peers can process real-time data, but the deployment and testing is significantly more complex.

From a code design perspective, each peer is comprised of a single Peer (class Protopeer.Peer) which in turn can contain an arbitrary number of Peerlets (class Protopeer.Peerlet). The peerlets offer services to the peer, such Bootstraping, PeerSampling, and other services as required by the application.


Peerlets
--------

The name for the peerlets is inspired by the applets and servlets. The peerlets encapsulate a well-defined piece of application functionality. Most of the application code lives inside the peerlets. Each peerlet instance has an associated peer instance in which it is contained. Several peerlets can be added together to a peer to achieve the desired functionality. The peer serves as an "execution context" for the peerlets. The peerlets can send messages using the peer's network interface and get notified about the networking events. Peerlets have access to the peer's clock, which they can use to schedule timers. The peerlets can also discover one another via the peer. For more see the peer documentation.

A peerlet looks like this:

.. code:: java

    public interface Peerlet {
        /* lifecycle */
        public void init(Peer peer);
        public void start();
        public void stop();

        /* networking */
        public void handleIncomingMessage(Message message);
        public void handleOutgoingMessage(Message message);
        public void messageSent(Message message);
        public void networkExceptionHappened(NetworkAddress remoteAddress, Message message, Throwable cause);

*Figure {x}* Interface for Peerlet class


*Lifecycle*

Just like applets, servlets or the peer, the peerlets have the init-start-stop- start-stop... lifecycle. When the Peer is inited, started or stopped it calls the init, start or stop on all of the peerlets that it contains. The peer argument in the init method lets the peerlet know which peer it's going to be associated with. Peerlet implementations implement these three methods with the peerlet-specific lifecycle handling code.

*Networking*

Whenever the peer receives a message it calls handleIncomingMessage on all of its peerlets. This method is where most of the message handling logic usually sits. The handleOutgoingMessage is called whenever someone calls sendMessage on the Peer, this notifies all the peerlets that some message is about to be sent. The messageSent methods is called when the network interface is done sending the message. Note, that this does not mean that the message was delivered at the destination. The peerlets are also notified about all the network exceptions via the networkExceptionHappened call. Peerlet implementations implement these networking methods with the peerlet-specific code.

BasePeerlet
-----------

To implement a peerlet you need to implement all of the methods in the Peerlet interface. Most of the time you don't put any code into all of these methods. For convenience, there is a BasePeerlet class that has blank implementations of all of these methods. In addition, there is the useful getPeer() methods that returns the peer in which the peerlet is contained.


