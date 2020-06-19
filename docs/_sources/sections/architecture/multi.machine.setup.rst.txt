Multi-machine Setup
*******************

DIAS is able to run the different components on different machines for splitting the computation power.
This is possible, as all communation between different processes is done via ZMQ, which uses TCP.

The parts being able to run separately are:

1. The different DIAS peers, including the carrier peers.
2. The persistence daemon, running the database
3. The device gateway
4. The bootstrap manager

The way this is done, is by changing the configuration files 'dias.conf' and 'protopeer.conf'.
Those files can be found under 'DIAS-Devlopment/deployments/<deployment-name>/conf/'.

The IP for the peers can be changed in 'dias.conf', at 'peerIP='.
The IP for the carriers is defined right below at 'peerZeroIP='

The other configurations are in the 'protopeer.conf' file, under the names 'persistenceDaemonIP', 'deviceGatewayPort', 'bootstrapIP'.

The config files must be consistent across all machines running any part of the setup.

**Note** If some of the machines do not receive anything, you may have to open the respective ports on the system, as well as on the router.

All of the ports are stated right below the correspoinding IP declaration in the config files.
One exception to this are the IP's for the DIAS peers. They dynamically allocate free ports, so the system needs to be able to dynamically allocate ports.
This setting may need to be done in the router settings of your router.