Command Flooder
***************


The command flooder experiment is a minimalistic example of a Protopeer-based network.

In this experiemtn, the network is comprised of a small number of peers, say 10. One of the peers is designated as the *captain*, typically peer 1.

Every few seconds, the captain will send a message called a *hot potato* to each of the peers connected to the *captain*. In turn, each of the peers that receive the *hot potato* will send it to the peers it is itself connected to.

And so on. The number of times the *hot potato* is pushed forward is defined by a parameter, set to 5 in the example below.


Launch Command
--------------

The following command will launch 10 independent peers in live mode, allowing messages to propagate 5 types through the network, using 256M Ram per peer

.. code:: bash

    ./command.flooder.zmq.sh 10 5 256M

*Code snippet: Example for launch 10 peers in the command flooder experiment*


You should that the number of peers in your Command Flooder network is greater or equal to the number of peers required for the Boostrapping.
