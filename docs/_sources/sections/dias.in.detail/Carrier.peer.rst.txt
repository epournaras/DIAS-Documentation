************
Carrier Peer
************

The carrier peer takes care of all disseminator of peers, which left the network.
It simulates normal operation by calling the active state of the disseminator, just as the peer itself did.
This way, the disseminator is still in contact with its neighbors, ensuring an up to date view of the network graph.

Once a peer rejoins the network, the carrier peer gets notified and returns the disseminator to the correct peer, deleting it from its own collection in the process.
