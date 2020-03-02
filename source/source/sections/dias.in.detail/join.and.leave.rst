**************
Join and Leave
**************

A peer is able to join and leave the running network.

Leave
#####

There is a graceful leave and a timeout leave.

Graceful Leave
**************

On a graceful leave, the device notifies the peer that it is leaving, the communication between the device and the peer is stopped until further notice.

Timout Leave
************

On a timeout leave, the peer leaves the network due to no received messages from the device over a predefined period of time.

Leaving Process
***************

When a node leaves the network, it stops aggregating and starts sleeping.
In order for the peer to still have information about the network and its neighbors, the node sends its Disseminator to a :ref:`Carrier Peer` before stopping its service.

Join
####

When a peer receives any message from a device, it reactivates itself, joining the network again in the process.
In order to rejoin, it notifies all carrier peers that it needs its disseminator back.
Upon receiving it's disseminator, it runs regularly again.
