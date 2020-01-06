Postgres
********

Although not strictly required, it is strongly recommended to install a Postgres database in order to record and plot the activity from the DIAS network. All of the documentation assumes that the DIAS Logging System has been setup.

At time of writing, Postgres is the only mechanism to record data generated within the DIAS network, through the :ref:`label_dias_logging_system`.

A variety of data is persisted to the database, such as:

- DIAS aggregates (sum, average, min, max, ...)

- Aggregation logs

- General free-text logging

- Structured logging

- Message exchange



Supported version of Postgres:

- 9.x

- 10.x


On Ubuntu, Postgres can be installed as follows:

.. code:: bash

    sudo apt-get install postgresql


On OSX, follow the instructions here https://www.postgresql.org/download/macosx/