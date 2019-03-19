DIAS-Documentation
edward | 2019-03-19

# ------------
# introduction
# ------------

DIAS stands for Dynamic Intelligent Aggregation Service

The purpose is to serve real-time data analytics, highly adaptive, truly decentralized. Made by citizens for citizens.

Dynamic
 - DIAS can operate in unreliable environments with rapidly changing input data
 
Intelligent
 - Thanks to an intelligent memory system that guarantees accurate measurements
 
Aggregation
 - Concerns the computation of several large-scale collective measurements
 
Service
 - DIAS can be used by different techno-socio-economic application

# ------------
# installation
# ------------

Installation is straight-forward and requires 3 steps

1. Setup the DIAS Logging System

2. Setup DIAS

3. For this demo, we will use the GDELT news feed, and thus will setup DIAS-GDELT

----------------------------------------------
Installation Step 1. Setup DIAS Logging System
----------------------------------------------

The output of the DIAS-GDELT system is stored in a PostgreSQL database, that must be setup in order for DIAS-GDELT to function correctly.

Installation of the database, that contains the output of the logging, is simple. The DIAS-Logging-System requries a running PostgreSQL database, that we will install in the first step just below.

1.1 Download the DIAS Logging System
git clone https://github.com/epournaras/DIAS-Logging-System


1.2 Install postgres database
* Note that this will also start the database service
# on an Ubuntu system (Ubuntu 14 or higher)
sudo apt-get update
sudo apt-get install postgresql


1.3 Create a database
* Login into PostgreSQL, using your favorite SQL editor or psql
* Choose any name, in this case we choose the name dias
    * If you change the database name, you will also need to update the file conf/daemon.conf accordingly
CREATE DATABASE dias;


1.4 Finally, create the tables that will store the logging information
* The source SQL can be found here:
    * DIAS-Logging-System/sql/definitions
        * eventlog.sql
        * memlog.sql
        * rawlog.sql
        * pss.sql
    * DIAS-Development/sql/definitions
        * aggregation.sql
        * aggregation_plot.sql
        * aggregation_event.sql
        * aggregation_event_rrd.sql
        * DIAS-visualization-db.sql
        * msgs.sql
        * sessions.sql

-------------------------------
Installation Step 2. Setup DIAS
-------------------------------

Here are the steps to install a generic version of DIAS.

git clone https://github.com/epournaras/DIAS-Development.git

git checkout pilot.2017.f

-------------------------------------
Installation Step 3. Setup DIAS-GDELT
-------------------------------------

We will now install the specific GDELT use case.

git clone https://github.com/epournaras/DIAS-GDELT

# ------
# launch
# ------

1 Launch the DIAS-Logging-System, by launching the persistence daemon (that listens for messages to be logged and writes them to the database).
cd DIAS-Logging-System
./start.daemon.sh deployments/gdelt

2 Launch Protopeer Bootstrap server and DIAS Gateway server
cd DIAS-Development
./start.servers.sh deployments/gdelt

3 Launch 28 DIAS Peers, one per country; no need for carrier nodes
cd DIAS-Development
./start.aggregation.peers.sh deployments/gdelt 28 1 1

4 Start GDELT subscription
cd DIAS-GDELT/python/gdeltv2.count
./auto.update.sh

4.5 start 28 GDELT Mock devices
cd DIAS-GDELT
 ./start.mock.devices.sh deployments/gdelt 28
