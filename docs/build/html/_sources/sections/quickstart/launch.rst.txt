.. _label_launch:

Launching DIAS
**************


Please check the :ref:`label_pre_requisites`, :ref:`label_quickstart_download` and :ref:`label_installation` sections before continuing.

We will assume a basic launch using default settings that are contained in the folder DIAS-Development/deployments/localhost.

DIAS with Logging
-----------------

If you installed the :ref:`label_dias_logging_system`, follow these instructions.

First, let's launch the DIAS Logging System.

.. code:: bash

    cd DIAS-Logging-System
    clear && ./start.daemon.sh deployments/localhost


Next, we need to start the Protopeer Bootstrap server and the DIAS Gateway server. More information can be found here :ref:`label_bootstrap_process` and :ref:`label_dias_gateway`.

.. code:: bash

    cd DIAS-Development
    ./start.servers.sh deployments/localhost



DIAS without Logging
--------------------

If you did not install the :ref:`label_dias_logging_system`, then follow these instructions.

