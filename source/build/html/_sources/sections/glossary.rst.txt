Glossary
********

.. list-table::
  :header-rows: 1

  * - Expression
    - Brief explanation
  * - DIAS
    - The name of the hereby described project, see :ref:`DIAS - Dynamic Intelligent Aggregation Service`
  * - DIAS node
    - One aggregating node inside the DIAS network
  * - GDELT
    - GDELT is the largest publicly available collection of near real-time news feed in the World, see :ref:`GDELT`
  * - Mock Device
    - A program simulating the input of real-world users for different experiments, see :ref:`Mock devices`
  * - Protopeer Framework
    - A platform written in Java for carrying out small to medium scale peer-to-peer experiments, see :ref:`Protopeer`
  * - DIAS Bootstrap server
    - The server, which initially interconnects the first peers in the DIAS network, see :ref:`Bootstrap Process`
  * - DIAS Gateway Server
    - The server connecting newly created peer to the already running network, see :ref:`DIAS Gateway`
  * - DIAS-Logging-System
    - The system, which allows the distributed system to log averything to a single database with relatively little contention, see :ref:`DIAS Logging`
  * - DIAS Network Id
    - The Network Id (oe sensor ID) is used to distinguish between different peers to be able to address a specific peer, see more here: :ref:`The usage of sensorIDs`,
  * - Possible states
    - The user needs to send a node different options, from which he later chooses a single one as a selected state. see :ref:`Changing of possible states`
  * - Selected state
    - The selected state is a selected element out of the possible states, it is considered as the current active value sent from a device. see :ref:`Changing of selected states`
  * - Peer Sampling Service
    - TODO
  * - Active state
    - Every DIAS node has an active state, it is basically an acttive thread running on a timer, repeatedly doing specific tasks.
  * - Passive state
    - Every DIAS node has a passive state, it gets triggered upon receiving a message, handling it corresponding to the defined actions.
  * - UNIX Screen
    - A UNIX command line tool, read |UNIXScreen|

.. |UNIXScreen| raw:: html

  <a href="https://kb.iu.edu/d/acuy" target="_blank">more</a>
