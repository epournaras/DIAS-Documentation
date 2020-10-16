************
Carrier Peer
************

The carrier peer takes care of all disseminator of peers, which left the network.
It simulates normal operation by calling the active state of the disseminator, just as the peer itself did.
This way, the disseminator is still in contact with its neighbors, ensuring an up to date view of the network graph.

Receiving a Disseminator
########################

The disseminator receives a leave message, containing the disseminator of the leaving node.
It lets the other collected disseminators aggregate with the newly received disseminator and adds it to its collection.

Sending a Disseminator Back
###########################

The disseminator regularly checks, if the DIAS nodes of its collected disseminators have come back online and set its return flag.
If so, it sends the disseminator back to its origin and removes it from the local collection.
