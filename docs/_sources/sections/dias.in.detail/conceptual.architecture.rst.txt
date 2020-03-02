***********************
Conceptual Architecture
***********************

Peerlets
########

DIAS is an intricate system with many parts, the most important are briefly explained in this section.

DIAS builds on :ref:`Protopeer`, which is modular and allows easy creation of submodules for a protopeer peer.
The submodules are called peerlets, the most important peerlets are:

* Aggregator
* Disseminator
* BootstrapGatewayClient
* RealTimeDiasApp

Aggregator
**********

The aggregator is responsible for the aggregation of the data distributed in the DIAS network, it is present in all aggregating DIAS peers.
There are multiple aggregates supported:

* average (=avg)
* sum
* maximum value (=max)
* minimum value (=min)
* the number uf currently aggregating (joined) peers (=count)

The aggregated data takes the selected state of all joined peers into consideration.
If a peer does not yet have a selected state, or it currently left the network, it will not affect the aggregates.

The aggregation process is thoroughly described in the section ":ref:`Aggregation Mechanism`"

Disseminator
************

The disseminator is responsible for the communication between different peers, it is present in all DIAS peers in the network.

To keep track of which peers were already notified about the own state, the Disseminator keeps track of those information in a counting bloom filter, allowing additions and removals, to keep the information up to date.
Read more about |bloomFilter|, :ref:`Aggregation Mechanism`

.. |BloomFilter| raw:: html

  <a href="https://en.wikipedia.org/wiki/Counting_Bloom_filter" target="_blank">Counting Bloom Filter</a>

Bloom Filters
*************

There are 4 different bloom filters used in DIAS for storing different memberships.



* | Aggregator j membership in disseminator i, called **AMD**
  |   Disseminator of peer i stores the identifier of the aggregator of peer j to which it has disseminated its own selected state at least once.
  |   This means this membership is present in a disseminator, if the aggregator in question has received the own selected state.
* | Aggregator j membership in a Possible state s of peer i, called **AMS**
  |   Disseminator of peer i stores the identifier of the aggregator of peer j for each possible state s aggregated by Aggregator j.
  |   This means this membership is present, if the selected state in question comes from the aggregator in question.
* | Disseminator i membership in Aggregator j, called **DMA**
  |   The aggregator of peer j stores the identifier of the Disseminator of peer i from which it has aggregated its selected state at least once.
  |   This means this membership is present, if the aggregator received the selected state of the disseminator in question and used it in its own aggregation.
* | Selected state s membership in aggregate j, called **SMA**
  |   The aggregator of peer j stores the identifier of a selected state s aggregated from a disseminator of a peer.
  |   This means this membership is present, if the aggregator used the selected state in question in its own aggregation.

AMD and AMS are present in the Disseminator, DMA and SMA are present in the Aggregator of a peer.

BootstrapGatewayClient
**********************

In the DIAS network, there is one peer with a the bootstrap peerlet.

This peerlet is responsible for interconnecting the first peers in the system.
It gets triggered once there are 10 peers in the system.
When triggered, it gives the 10 peers a view of some of the other peers in the network, such that they can start communicating in the network.

Read more about the :ref:`Bootstrap Process`

ZeroMQReqRepListener
********************

This peerlet is in all aggregating peers, it keeps the connection between the peer and the connected device, it demultiplexes all incoming messages and notifies the correspndong correct peerlets.

**Note:**
  | The messages are only checked, if the sender is the initially paired sender.
  | The first time the peerlet receives a message from a device, the pairing is saved.

The following messages can be interpreted by this peerlet:

.. list-table::
  :header-rows: 1

  * - Message
    - Action
  * - LeaveMessage
    - Tells the peer to leave the network
  * - PingMessage
    - Asks the peer to send a PongMessage
  * - HearbeatMessage
    - Sends a hearbeat, this message itself does not trigger anything, but it ensures that the peer does not time out.
  * - SensorCommandMessage
    - This message is at the moment only used to set the timeout threshold of the peer to a desired value.
  * - AggregateRequestMessage
    - This message requests the current aggregate from the peer, the peer answers with the requested value, or with a nok message, if the aggregate is not available.
      The following aggregates are available (see :ref:`Aggregator`):

      * avg
      * sum
      * max
      * min
      * count
  * - StartAggregationMessage
    - This message triggers the initial start for aggregation, before this message is received, the peer is offline and is not aggregating.
  * - StopAggregationMessage
    - This message stops the aggregation and leaves the network
  * - PossibleStatesMessage
    - This message sets the possible states to the given values, see :ref:`Changing of possible states`
  * - SelectedStateMessage
    - This message sets the selected state to the given values under some constraints, see :ref:`Changing of selected states`
