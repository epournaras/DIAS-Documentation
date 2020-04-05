.. _label_bootstrap_process:

Bootstrap Process
*****************

The bootstrap process initialises the connection between the initial peers in the network.
In DIAS, the bootstrap process starts as soon as 10 peers are online in the network.

During the process, the bootstrap peer sends an initial set of neighbors to all peers.
The elements in those set can be arbitrary, but the network must not be separated.
in DIAS, the network graph admits a star shape with two peers in the center.

Once the initial setup is done, the bootstrap node gives each additionally joining peer a subset of the view of the
already created network.

The initial view of the peers will however not remain the only neigbors, as the peers interchange their view with their neighbors.

The bootstrap process is needed, as initially joining peers have no idea of the network and need a starting view,
it ensures that the network initially is not separated.
