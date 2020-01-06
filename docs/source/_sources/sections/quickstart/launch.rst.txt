.. _label_launch:

Launching DIAS
**************


Please check the :ref:`label_pre_requisites`, :ref:`label_quickstart_download` and :ref:`label_installation` sections before continuing.

We will assume a basic launch using default settings that are contained in the folder DIAS-Development/deployments/localhost.

If you installed the :ref:`label_dias_logging_system`, follow these instructions.

First, let's launch the DIAS Logging System. We recommend launching each of the below in a separate shell window to facilitate monitoring.

.. code:: bash

    # start persistence daemon, used to store aggregates and debugging information to Postgres

    cd DIAS-Logging-System
    clear && ./start.daemon.sh deployments/localhost


Next, we need to start the Protopeer Bootstrap server and the DIAS Gateway server. More information can be found here :ref:`label_bootstrap_process` and :ref:`label_dias_gateway`.

.. code:: bash

    # start Protopeer server and Gateway server

    cd DIAS-Development
    ./start.servers.sh deployments/localhost

Now we can start some carrier peers. Carrier peers allows peers to leave the network for short or long periods of time by performing corrective operations so that the network maintains correct aggregates.
You can launch as many carrier nodes as you wish, 2 is a good start on a small networks.

.. code:: bash

    # launch 2 carrier peers

    cd DIAS-Development
    ./start.carrier.peers.sh deployments/localhost 2


We can then launch the DIAS aggregation peers. These are the peers that are actually performing the aggregations. In the example below, we launch 10 peers. You can launch upto 40 at a time.

We also specify the id of the first DIAS aggregation peer, in this case 3. This is because peer 0 is the Protopeer bootstrap server and peers 1 and 2 are the carrier nodes that were launched above.

.. code:: bash

    # start 10 aggregation peers, starting at peer id 3

    cd DIAS-Development
    ./start.aggregation.peers.sh deployments/localhost 10 3

Finally, we launch the mock devices. Mock devices simulate real IoT devices such has hand-held mobile phones, sensors, etc


.. code:: bash

    # launch 10 mock devices, starting at mock device number 3

    cd DIAS-Development
    ./start.mock.devices.sh deployments/localhost 10 3

