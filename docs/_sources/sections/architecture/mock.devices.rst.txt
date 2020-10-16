Mock devices
************

The mock devices act as device users and simulate desired behaviour in a specific controlled way.

A mock device can basically defined in any desired way, the following code snippet is an example how a mock device can look like.

.. code-block:: java
  :linenos:

  public class ExampleMockDevice
  {
    // send a message to a socket; returns the number of bytes sent
    private static int sendMsg(ZMQ.Socket socket, String msg){
      final int strlen = msg.length();
      ByteBuffer bb = ByteBuffer.allocate(strlen).order(ByteOrder.nativeOrder());
      bb.put(msg.getBytes(ZMQ.CHARSET));
      bb.flip();
      final int ret = socket.sendByteBuffer(bb, 0);
      return ret;
    }

    public static void main(String[] args) throws InterruptedException {
      if(args.length < 5) {
        System.out.printf("usage: device.id gateway.port gateway.host seed.ms num.peers\n");
        return;
      }

      // random number generator
      final Random randomNumbers = new Random();

      // constants
      final int device_id = Integer.parseInt(args [0]),
      gatway_port = Integer.parseInt(args [1]);

      // description about this peer
      SensorAgentDescription sensorAgent = new SensorAgentDescription("dev#" + device_id, 1);
      SensorDescription sensorDescription = new SensorDescription("SensorA", "GPS");

      // paths
      final String host = args [2];

      // --------
      // settings
      // --------

      final boolean selectedStateIsAlwaysZero  = false;
      final boolean deviceChangesSelectedState = true;
      final int stepsSelectedState             = 10;    // number of steps to wait until sending a selected state (if sendSelectedStates is true)
      final int numChangesSelectedState        = 20000; // number of times the devices that change state should change (only has an effect if deviceChangesSelectedState is true)
      final int numDimensions                  = 1;     // number of scalars per state

      // ------------------------
      // message sending behavior
      // ------------------------

      final boolean debug = false,
      sendHeartbeats = true,
      sendPeerTimeoutMsg = true;                                               // if true, sends the value of the timeout threshold to the peer
      final int stepDurationMs = 1000,                                         // duration of a step in milli-seconds
                stepsHeartbeat = 1 + randomNumbers.nextInt(20),                // number of steps to wait until sending a heartbeat (if sendHeartbeats is true)
                peerTimeoutMs  = 5000 + (2 * stepsHeartbeat * stepDurationMs); // allow a base of 5 seconds, then proportional to the heartbeat frequency

      // list of the possible states
      ArrayList<SensorState> possibleStates = new ArrayList<SensorState>();

      // continue with initialisation
      SimpleDateFormat dateFormatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

      // zeromq context; give 2 threads
      // note that there should only be a single zeromq context per executable
      ZMQ.Context zmqContext = ZMQ.context(2);

      // message serialisation
      Gson gson = new Gson();

      // factory for converting DIAS GUI messages from JSON
      MessageFactory factory = new MessageFactory();

      // ------------------
      // connect to gateway
      // ------------------

      String peerFinger = "";;

      Integer peerId = -1;

      final String gatewayConnectString = "tcp://" + host + ":" + gatway_port;

      // connect
      System.out.printf("connecting to gateway %s...", gatewayConnectString);
      ZMQ.Socket gateway_socket= zmqContext.socket(ZMQ.REQ);
      gateway_socket.connect(gatewayConnectString);
      gateway_socket.setRcvHWM(1000);
      gateway_socket.setLinger(-1);
      System.out.printf("ok\n");

      // create PeerAddressRequest
      PeerAddressRequestMessage peerAddressRequestMessage = new PeerAddressRequestMessage(sensorAgent);
      String peerAddressRequestMessageJson = gson.toJson(peerAddressRequestMessage);
      System.out.printf("peerAddressRequestMessageJson : %s\n", peerAddressRequestMessageJson);

      // send message
      System.out.printf("sending address request to gateway...");
      sendMsg(gateway_socket, peerAddressRequestMessageJson);
      System.out.printf("ok\n");

      // wait for response
      System.out.printf("\nwaiting for response...");
      String gatewayResponse = new String(gateway_socket.recv(), ZMQ.CHARSET);
      System.out.printf("ok\n");

      if(debug) System.out.printf("incomingMessage : %s\n", gatewayResponse);

      // disconnect from gateway
      System.out.printf("disconnecting from gateway...");
      gateway_socket.close();
      System.out.printf("ok\n");

      gateway_socket = null;

      // verify that the gateway connected us to a peer
      GUIMessage guiMessage = factory.CrackMessage(gatewayResponse);
      System.out.printf("guiMessage : %s\n", guiMessage.toString());

      String mtype = guiMessage.MessageType;
      System.out.printf("mtype : %s\n", mtype);

      switch(mtype) {
        case "Ack":
          AckMessage ackMessage = gson.fromJson(gatewayResponse, AckMessage.class);
          System.out.printf("received an ACK message with msg %s | %s -> goodbye\n", ackMessage.ackText, ackMessage.errorText);
          return;

        case "PeerAddress":
          PeerAddressMessage peerAddressMessage = gson.fromJson(gatewayResponse, PeerAddressMessage.class);
          peerFinger = peerAddressMessage.peerFinger;
          peerId = peerAddressMessage.peerId;
          System.out.printf("peerId : %s\n", peerId);
          System.out.printf("peerFinger : %s\n", peerFinger);
          System.out.printf("\n*** Will bind with peer %d (%s) ***\n", peerAddressMessage.peerId, peerFinger);
          break;

        default:
          System.out.printf("unhandled mtype : %s\n", mtype);
      }

      // -----------------------
      // --- connect to peer ---
      // -----------------------

      // connect to the peer
      final String	peerConnectString = "tcp://" + peerFinger;

      // connect to peer
      ZMQ.Socket zmqPeerSocket = null;
      System.out.printf("connecting to peer #%d on %s...", peerId, peerConnectString);

      zmqPeerSocket = zmqContext.socket(ZMQ.REQ);
      zmqPeerSocket.connect(peerConnectString);
      zmqPeerSocket.setHWM(100);
      zmqPeerSocket.setLinger(-1);
      System.out.printf("ok\n");

      // send our own internal timeout message
      if(sendPeerTimeoutMsg) {
        SensorSetControlMessage setPeerTimeoutMessage = new SensorSetControlMessage(sensorAgent,
                                                                                    sensorDescription,
                                                                                    "TimeoutMs",
                                                                                    Integer.toString(peerTimeoutMs)
                                                                                   );

        System.out.printf("setPeerTimeoutMessage : %s\n", gson.toJson(setPeerTimeoutMessage));

        System.out.printf("sending TimeoutMs message...");


        sendMsg( zmqPeerSocket, gson.toJson(setPeerTimeoutMessage));
        System.out.printf("ok\n");

        // wait for response
        System.out.printf("waiting for response...");

        final String response = new String(zmqPeerSocket.recv(), ZMQ.CHARSET);
        if(!debug)
          System.out.printf("ok\n");
        else
          System.out.printf("<- %s\n", response);
      }

      // connection complete
      System.out.printf("\n### CONNECTED ###\n");
      System.out.printf("%s\n", dateFormatter.format( System.currentTimeMillis()));
      System.out.printf("sensorAgent : %s\n", sensorAgent);

      // ----------------------
      // create possible states
      // ----------------------

      possibleStates.clear();

      System.out.printf("P");

      if(selectedStateIsAlwaysZero) {
        // possible states are all zero -> selected state is also zero

        // create the means that represent the cluster
        LinkedHashMap<String,Object> cluster_means = new LinkedHashMap<String,Object>();

        // 0.0
        cluster_means.put("x.1", 0.0);
        SensorState state = new SensorState( new Integer(0), cluster_means);
        possibleStates.add( state );

        System.out.println("Single state 0 created");
      }
      else {
        // 3
        LinkedHashMap<String,Object> cluster_means0 = new LinkedHashMap<String,Object>();
        cluster_means0.put("x.1", 3.0);
        SensorState state0 = new SensorState( new Integer(0), cluster_means0 );
        possibleStates.add(state0);

        // 5
        LinkedHashMap<String,Object> cluster_means1 = new LinkedHashMap<String,Object>();
        cluster_means1.put("x.1", 5.0);
        SensorState state1 = new SensorState( new Integer(1), cluster_means1);
        possibleStates.add(state1);

        // 7
        LinkedHashMap<String,Object> cluster_means2 = new LinkedHashMap<String,Object>();
        cluster_means2.put("x.1", 7.0);
        SensorState state2 = new SensorState(new Integer(2), cluster_means2);
        possibleStates.add(state2);

        System.out.println("States 3, 5 and 7 created");
      }

      final int numPossibleStates = possibleStates.size();
      System.out.println("numPossibleStates: " + numPossibleStates);

      // -------------------
      // send possible states
      // -------------------
      PossibleStatesMessage possibleStatesMessage = new PossibleStatesMessage(possibleStates, sensorAgent, sensorDescription);

      sendMsg(zmqPeerSocket, gson.toJson(possibleStatesMessage));

      System.out.println("Possible states sent");

      // wait for response
      String response = new String(zmqPeerSocket.recv(), ZMQ.CHARSET);
      System.out.println("response : " + response);

      Thread.sleep(stepDurationMs);

      // ---------------------------
      // send initial selected state
      // ---------------------------

      // all peers start with the first state
      int selectedStateId = 0;
      System.out.printf("Initial Selected State Id: %s\n", selectedStateId);

      // generate selected state message
      SensorState selectedState = possibleStates.get(selectedStateId);

      SelectedStateMessage selectedStateMsg = new SelectedStateMessage(selectedState, sensorAgent,sensorDescription);

      sendMsg(zmqPeerSocket, gson.toJson(selectedStateMsg));
      System.out.println("Initial selected state sent");

      // wait for response
      response = new String(zmqPeerSocket.recv(), ZMQ.CHARSET);
      System.out.println("response : " + response);

      Thread.sleep(stepDurationMs);

      // -----------------
      // start aggregation
      // -----------------

      sendMsg(zmqPeerSocket, gson.toJson(new StartAggregationMessage(sensorAgent)));
      System.out.println("Start Aggregation sent");

      // wait for response
      response = new String(zmqPeerSocket.recv(), ZMQ.CHARSET);
      System.out.println("response : " + response);

      Thread.sleep(stepDurationMs);

      // -------------------------
      // send messages to the peer
      // -------------------------

      long step = 0l;

      int selectedStateChangeCount = 0;

      // main loop; infinite
      while(true){
        ++step;

        Thread.sleep(stepDurationMs);

        ArrayList<String> messagesToSend = new ArrayList<String>();

        // -----------------------
        // send new selected state
        // -----------------------
        if(deviceChangesSelectedState && (selectedStateChangeCount < numChangesSelectedState) && ((step % stepsSelectedState) == 0)) {
          selectedStateId = randomNumbers.nextInt(numPossibleStates);

          // on the last programmed changed in selected state, return to the initial value
          if((selectedStateChangeCount+1) == numChangesSelectedState) {
            selectedStateId = 0;
            System.out.println("\nlast selected state");
          }

          if(debug) System.out.printf("selectedStateId : %s\n", selectedStateId);

          if(debug)
            System.out.printf("\nSending inital selected state %d since sending selected states is disabled\n", selectedStateId);
          else
            System.out.printf("\n%d / %d : S(" + possibleStates.get(selectedStateId).stateValues.toString() + ")", selectedStateChangeCount, numChangesSelectedState);

          // generate selected state message
          selectedState = possibleStates.get(selectedStateId);
          if(debug) System.out.printf("selectedState : %s\n", selectedState);

          selectedStateMsg = new SelectedStateMessage(selectedState, sensorAgent,sensorDescription);
          if(debug) System.out.printf("selectedStateMsg : %s\n", selectedStateMsg);

          // add as a JSON string to the list of messages to send
          messagesToSend.add(gson.toJson(selectedStateMsg));

          ++selectedStateChangeCount;
        }

        // --------------
        // send heartbeat
        // --------------

        if(sendHeartbeats && ((step % stepsHeartbeat) == 0)) {
          // generate heartbeat message
          HeartbeatMessage heartbeatMessage = new HeartbeatMessage(sensorAgent, sensorDescription);

          // add as a JSON string to the list of messages to send
          messagesToSend.add(gson.toJson(heartbeatMessage));

          System.out.printf("H");
        }

        // -------------
        // send messages
        // -------------

        for(String msg:messagesToSend) {
          // send msg to peer
          sendMsg(zmqPeerSocket, msg);
          if(debug) System.out.printf("ok\n");

          // wait for response
          if(debug) System.out.printf("waiting for response...");

          response = new String(zmqPeerSocket.recv(), ZMQ.CHARSET);
          if(debug) System.out.printf("ok : %s\n", response);
        }// next msg
      }// while (main loop)
    }// main
  }// Server

This Mock device changes its possible state every 10 seconds.
