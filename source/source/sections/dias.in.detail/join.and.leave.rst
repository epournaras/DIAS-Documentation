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
If the DIAS node is currently joined in the network, it sets a flag, such that it leaves in the next iteration. 
If the node is currently not joined, the device communication handler ignores the request and notifies the requester, that the node has already left the system.

Tiemout Leave
*************

On a timeout leave, the peer leaves the network due to no received messages from the device over a predefined period of time.
Every time the device communication handler receives a message from the connected device, it saves the timestamp in the DIAS node.
In every iteration of the node, it checks, if the last received message exceeds a specified threshold.
If this is the case, the node sets a flag to leave in the next iteration.

Leaving Process
***************

When a node leaves the network, it stops aggregating and starts sleeping. This happens when the leave flag is set.
In order for the peer to still have information about the network and its neighbors, the node sends its Disseminator to a :ref:`Carrier Peer` before stopping its service.
For this, it selects a random carrier peer of it's list of available carriers.
It then sends it's disseminator to this selected carrier peer and stops all it's actions.

If the node leaving is a carrier, it distributes all it's collected disseminators to other carrier peers before leaving.

Join
####

When a peer receives any message from a device, it reactivates itself, joining the network again in the process.
In order to rejoin, it notifies all carrier peers that it needs its disseminator back.
Since the disseminator could have changed carrier since the node left the network, it cannot directly notify the carrier carrying the disseminator, as it does not know which carrier has it.
Instead, it sets a flag with a UUID, which matches the one the disseminator carries.
When the carriernotices this flag, it will then send back the matching disseminator to the DIAS node.
Since most of the operations of a DIAS node need a working disseminator, the node cannot return to normal operation before successfully receiving said disseminator.
Because of that, the node waits for the disseminator, before starting itself up.
After that, the flag gets removed again, and the node returns to normal operation.

Special Cases
#############

* **The peer wants to leave the network and thus send it's disseminator to a carrier peer, but it seems to have no disseminator**

This case can only happen after rejoining the network, before the node received its disseminator back from a carrier.
The node simply skips the whole leaving process, but leaves the leave flag set. 
This way, it will try to leave again in the next iteration, when the disseminator hopefully returned.


* **The peer wants to leave the network and thus send it's disseminator to a carrier peer, but it can't find any carrier peers**

This case should only happen after rejoining the network, before it hase enough knowledge of its neighbors and the carrier peers within.
When handling a leave request in the device communication manager, it waits for carrier peers do appear in the view of the DIAS node, before notifying it to leave.
If the leave is triggered by a timeout, it should always have a carrier in its view and should thus not be a problem for the system.
If this case would still happen, and the DIAS node is trying with no carriers in its view, it stops the peer, such that the last logging information is from the malicious state.


* **The peer waits for its disseminator, but then actually receives it twice**

It simply ignores the second disseminator, as it already has received it before.