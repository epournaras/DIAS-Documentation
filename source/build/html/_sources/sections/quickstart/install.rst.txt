.. _label_installation:

Installation + Setup 
********************

DIAS requires no further installation other than the required downloads (:ref:`label_quickstart_download`).

Deployments
-----------

DIAS comes with pre-configured deployments. As there are a number of components required to operate a DIAS network (e.g :ref:`label_protopeer_overview`), there are various configuration files for each component. For convenience, these are all grouped into components.

In the next section, when you launch DIAS (:ref:`label_launch`), you will need to choose a deployment.

To get started, you should choose deployments/localhost, but you may add or modify deployments.



SQL Setup
---------

The following tables need to be created in the Postgres database. 

The first step is to create the DIAS database itself. Within psql (or any other Postgres editor):

.. code:: bash

    CREATE DATABASE dias;

Then you need to switch to the dias database; recall that when you login to psql, the default database is postgres.

.. code:: bash

    \c dias

Now we can create the tables that store the aggregates and debugging information.

The SQL code with the table definitions is located here. Copy + paste the code from each file, one at a time.

- DIAS-Development/sql/definitions/Logging-System/eventlog.sql
- DIAS-Development/sql/definitions/Logging-System/memlog.sql
- DIAS-Development/sql/definitions/Logging-System/rawlog.sql
- DIAS-Development/sql/definitions/Logging-System/pss.sql

- DIAS-Development/sql/definitions/DIAS/aggregation.sql
- DIAS-Development/sql/definitions/DIAS/aggregation_plot.sql
- DIAS-Development/sql/definitions/DIAS/aggregation_event.sql
- DIAS-Development/sql/definitions/DIAS/aggregation_event_rrd.sql
- DIAS-Development/sql/definitions/DIAS/aggregation_event_rrd_7d.sql
- DIAS-Development/sql/definitions/DIAS/msgs.sql
- DIAS-Development/sql/definitions/DIAS/sessions.sql

- DIAS-Development/sql/definitions/Protopeer/msgqueue.sql
- DIAS-Development/sql/definitions/Protopeer/msgtypes.sql
