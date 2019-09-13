.. highlight:: java
  :linenothreshold: 6

Aggregation Mechanism
*********************


Case 1
------

Conditions:

    - Both peers have communicated before

    - Thus positiveAMD and positiveDMA are both true

.. .. image:: aggregation.mechanism/case.1.png

.. **Figure** DIAS Aggregation Case 1

::

  if(positiveAMD && positiveDMA)
    {
      // Level 1.1: Check for false positives in the positive AMS memberships
      HashSet<State> posAMS = (HashSet) disseminatorReport.get(DisseminatorReport.POSITIVE_AMS);
      Iterator<State> it = posAMS.iterator();
      logEvent( "receiveDisseminatorReport", "case", "1" );
      this.log(1,"1. CURRENT STATE { "	+
                  " positiveAMD = "		+	positiveAMD		+	" | positiveDMA = "	+ positiveDMA	+
                " | shouldRemove = "	+ 	shouldRemove	+	" | posAMS.size() =  " 	+ posAMS.size() +
                " | purpose = " 		+ 	predictedOutcome+ 	" }");

      while(it.hasNext()){
        State state = it.next();
        if(!this.SMA.contains(state.getStateId().toString())) {
          if(this.SMA.contains(newState.getStateId().toString())) {
            tobeRemoved.add(state);
          }
          it.remove();
          this.log(1, "REMOVED STATE: " + state + ", NEW STATE IS: " + newState + ", SMA contains new state: " + this.SMA.contains(newState.getStateId().toString()));
        }
      }

+------------------+---------------------------------------------------------------------------+
| Line number      | Description                                                               |
+==================+===========================================================================+
| line 1           | 1. Peers have communicated before (start time = Big Bang)                 |
+------------------+---------------------------------------------------------------------------+
| line 12          | Iterate on all the states that the "sending peer" thinks that I'm using   |
+------------------+---------------------------------------------------------------------------+
| line 14          | 2. Disagree! I am not using this state in my aggregate                    |
+------------------+---------------------------------------------------------------------------+
| line 19          | | This will set th AMSRemoval field in the PULL message returned to the   |
|                  | | "sending peer" in order to update his own AMSs so that the sending peer |
|                  | | can remove this state for this peer Dissemeniot:receiveAggregatorReport |
+------------------+---------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:1
  | positiveDMA:1
  | SMA[posAMS] = false
  | SMA[selected_state] = true

Case 2
------

.. .. image:: aggregation.mechanism/case.2.png

.. **Figure** DIAS Aggregation Case 2

.. code-block:: java
  :linenos:

  if(positiveAMD && positiveDMA)
  {
    if(posAMS.size() == 1)
    {
      State oldState = posAMS.iterator().next();
      if(shouldRemove) {
        AMSRemoval = oldState;
        AMSAddition = null;
        boolean removed = this.removeSMAMembership(oldState);
        if(!removed) {
          if(possibles != null) {
            Iterator<State> iterator = possibles.iterator();
            boolean a = false;
            while(iterator.hasNext()) {
              State state = iterator.next();
              if(this.removeSMAMembership(state)) {
              	this.removeAggregationState(state);
              	AMSRemoval = state;
              	outcome = AggregationOutcome.REMOVAL;
                ack = true;
                a = true;
                this.log(1,"#####################");
                break;
              }
            }
            if(a == false)
            {
              logEvent( "receiveDisseminatorReport", "case", "2" );
              this.log(1, "2. CURRENT STATE { " +
                    " positiveAMD = " 		+ positiveAMD 		+ 	" | positiveDMA = "		+ 	positiveDMA +
                    " | shouldRemove = " 		+ shouldRemove 		+ 	" | posAMS.size() = " 	+ 	" 1 " 		+
                    " | purpose = " 			+ predictedOutcome 	+ 	" | removed = " 		+ 	removed 	+
                    " | possibles == null ? "	+ (possibles==null)	+ 	" | NOTHING REMOVED }");
            }


+------------------+----------------------------------------------------------------------------------------+
| Line number      | Description                                                                            |
+==================+========================================================================================+
| line 1           | 1. Peers have communicated before (start time = Big Bang)                              |
+------------------+----------------------------------------------------------------------------------------+
| line 3           | 2. Only a single state in the list of all states that the sending peers                |
|                  |    think that we have                                                                  |
+------------------+----------------------------------------------------------------------------------------+
| line 6           | 3. Other peer thinks we should remove it (recall that there is only 1)                 |
|                  |    strategy = REMOVAL                                                                  |
+------------------+----------------------------------------------------------------------------------------+
| line 9           | Try to remove it                                                                       |
+------------------+----------------------------------------------------------------------------------------+
| line 10          | Didn't find the state to be removed in my SMAs                                         |
+------------------+----------------------------------------------------------------------------------------+
| line 14          | Iterate throug all possible states of the sending peer                                 |
+------------------+----------------------------------------------------------------------------------------+
| line 17          | Remove it from the SMA                                                                 |
+------------------+----------------------------------------------------------------------------------------+
| line 19          | | This will set th AMSRemoval field in the PULL message returned to the "sending peer" |
|                  | | in order to update his own AMSs so that the sending peer can remove this state for   |
|                  | | for this peer Dissemeniot:receiveAggregatorReport                                    |
+------------------+----------------------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:1
  | positiveDMA:1
  | posAMS.size:1
  | purpose:REMOVAL*
  | SMA[posAMS] = false
  | POS :sup:`1`: *not null*
  | SMA[selected_state] :sup:`2` = true


**Notes**

  | * shouldRemove:true
  | 1 POS: possible states of the sending peer
  | 2 SMA[POS]: SMA contains one state from the possible states of the sending peer
  | newState = DisseminatorReport.SELECTED_STATE

Case 3
------

.. .. image:: aggregation.mechanism/case.3.png

.. **Figure** DIAS Aggregation Case 3

.. code-block:: java
  :linenos:

  if(positiveAMD && positiveDMA)
  {
    if(posAMS.size() == 1)
      {
        State oldState = posAMS.iterator().next();
        if(shouldRemove) {
          AMSRemoval = oldState;
          AMSAddition = null;
          boolean removed = this.removeSMAMembership(oldState);
          if(!removed) {
            if(possibles != null) {
              // not executed in this case, omitted here
            } else
            {
              logEvent( "receiveDisseminatorReport", "case", "3" );
              this.log(1, "3. CURRENT STATE { " +
                    " positiveAMD = " 		+ positiveAMD 		+ 	" | positiveDMA = " 	+ 	positiveDMA +
                    " | shouldRemove = " 		+ shouldRemove 		+ 	" | posAMS.size() = " 	+ 	" 1 " 		+
                    " | purpose = " 			+ predictedOutcome 	+ 	" | removed = " 		+ 	removed 	+
                    " | possibles == null ? " 	+ (possibles==null) + 	" | NOTHING REMOVED }");
            }

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 1           | 1. Peers have communicated before (start time = Big Bang)               |
+------------------+-------------------------------------------------------------------------+
| line 3           | 2. Only a single state in the list of all states that the sending peers |
|                  |    think that we have                                                   |
+------------------+-------------------------------------------------------------------------+
| line 6           | 3. Other peer thinks we should remove it (recall that there is only 1)  |
|                  |    strategy = REMOVAL                                                   |
+------------------+-------------------------------------------------------------------------+
| line 9           | Try to remove it                                                        |
+------------------+-------------------------------------------------------------------------+
| line 10          | Oups! Didn't find the state to be removed in my SMAs!                   |
+------------------+-------------------------------------------------------------------------+
| line 11          | Sending peer has no possible states                                     |
|                  | (Just to avoid null pointer exceptions)                                 |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:1
  | positiveDMA:1
  | posAMS.size:1
  | purpose:REMOVAL*
  | SMA[posAMS] = false
  | POS :sup:`1`: *null*
  | SMA[selected_state] :sup:`2` = true


  **Notes**

    | * shouldRemove:true
    | 1 POS: possible states of the sending peer
    | 2 SMA[POS]: SMA contains one state from the possible states of the sending peer
    | newState = DisseminatorReport.SELECTED_STATE


Case 4
------

.. .. image:: aggregation.mechanism/case.4.png

.. **Figure** DIAS Aggregation Case 4

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    if(posAMS.size() == 1)
    {
      State oldState = posAMS.iterator().next();
      if(shouldRemove) {
      	AMSRemoval = oldState;
      	AMSAddition = null;
        boolean removed = this.removeSMAMembership(oldState);
        if(!removed) {
          // Not executed in case 4, omitted
        } else
        {
          this.removeAggregationState(oldState);

          logEvent( "receiveDisseminatorReport", "case", "4" );
          this.log(1,"4. CURRENT STATE { " +
                " positiveAMD = " 		+ positiveAMD 		+ 	" | positiveDMA = " 	+ 	positiveDMA +
                " | shouldRemove = " 		+ shouldRemove 		+ 	" | posAMS.size() = " 	+ 	" 1 " 		+
                " | purpose = " 			+ predictedOutcome 	+ 	" | removed = " 		+ 	removed 	+
                " | possibles == null ? " 	+ (possibles==null) + 	" | OUTCOME = REMOVAL }");
          outcome=AggregationOutcome.REMOVAL;
          ack=true;
        }
      }

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 1           | 1. Peers have communicated before (start time = Big Bang)               |
+------------------+-------------------------------------------------------------------------+
| line 3           | 2. Only a single state in the list of all states that the sending peers |
|                  |    think that we have                                                   |
+------------------+-------------------------------------------------------------------------+
| line 6           | 3. Other peer thinks we should remove it (recall that there is only 1)  |
|                  |    strategy = REMOVAL                                                   |
+------------------+-------------------------------------------------------------------------+
| line 9           | Try to remove it                                                        |
+------------------+-------------------------------------------------------------------------+
| line 12          | Successfully removed it                                                 |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:1
  | positiveDMA:1
  | posAMS.size:1
  | purpose:REMOVAL*
  | SMA[posAMS] = false


**Notes**

  | * shouldRemove:true
  | 1 POS: possible states of the sending peer
  | 2 SMA[POS]: SMA contains one state from the possible states of the sending peer
  | newState = DisseminatorReport.SELECTED_STATE


Case 5
------

.. .. image:: aggregation.mechanism/case.5.png

.. **Figure** DIAS Aggregation Case 5

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    if(posAMS.size() == 1)
    {
      State oldState=posAMS.iterator().next();
      if(shouldRemove) {
        // not executed in case 5, omitted
      }
      else
      {
        if(!oldState.equals(newState))
        {
          double count0 = (Double) this.aggregates.getAggregate(AggregationFunction.COUNT);
          AMSRemoval = oldState;
          boolean s1 = this.removeSMAMembership(oldState);
          this.removeAggregationState(oldState);
          double count1 = (Double)this.aggregates.getAggregate(AggregationFunction.COUNT);
          AMSAddition = newState;
          boolean s2 = this.addSMAMembership(newState);
          this.addAggregationState(newState);
          double count2 = (Double) this.aggregates.getAggregate(AggregationFunction.COUNT);

          logEvent( "receiveDisseminatorReport", "case", "5" );
          this.log(1,"5. CURRENT STATE { " +
                " positiveAMD = " 			+ positiveAMD 		+ " | positiveDMA = " 					+ 	positiveDMA 				+
                " | shouldRemove = " 			+ shouldRemove 		+ " | posAMS.size() = " 				+ 	" 1 " 						+
                " | purpose = " 				+ predictedOutcome 	+ " | old.State.equals(newState) = "	+ 	oldState.equals(newState) 	+
                " | removeOldSMAMembership = " 	+ s1 				+ " | addNewSMAMembership = " 			+ 	s2 							+
                " | count0 = "					+ count0			+ " | count1 = "						+ 	count1						+
                " | count2 = "					+ count2			+
                " | OUTCOME = UPDATE }");
          outcome = AggregationOutcome.REPLACE;
          ack = true;

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 1           | 1. Peers have communicated before (start time = Big Bang)               |
+------------------+-------------------------------------------------------------------------+
| line 3           | 2. Only a single state in the list of all states that the sending peers |
|                  |    think that we have                                                   |
+------------------+-------------------------------------------------------------------------+
| line 9           | 3. Other peer thinks we should EXPLOIT it                               |
|                  |    (recall that there is only 1 state in posAMS)                        |
+------------------+-------------------------------------------------------------------------+
| line 11          | posAMS[0] != new selected state                                         |
+------------------+-------------------------------------------------------------------------+
| line 16 - 20     | Replace the old selected state with the new one                         |
+------------------+-------------------------------------------------------------------------+
| line 32          | Successfully replaced it                                                |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:1
  | positiveDMA:1
  | posAMS.size:1
  | purpose:EXPLOITATION*
  | posAMS[0] != SELECTED_STATE


**Notes**

  | * shouldRemove:false
  | newState = DisseminatorReport.SELECTED_STATE


Case 6
------

.. .. image:: aggregation.mechanism/case.6.png

.. **Figure** DIAS Aggregation Case 6

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    if(posAMS.size() == 1)
    {
      State oldState = posAMS.iterator().next();
      if(shouldRemove) {
        // Not executed in case 6, omitted here
      }
      else
      {
        if(!oldState.equals(newState)){
          // Not executed int case 6, omitted here
        }
        else
        {
          try{
            this.aggregates.updateMaxMin(newState);
          }
          catch(StateException ex)
          {
            RawLog.print(1,ex.toString() + ex.getStateExcMsg());
          }

          logEvent( "receiveDisseminatorReport", "case", "6" );
          this.log(1,"6. CURRENT STATE { " +
                " positiveAMD = " 	+ positiveAMD 		+ 	" | positiveDMA = " 				+ 	positiveDMA 				+
                " | shouldRemove = " 	+ shouldRemove 		+ 	" | posAMS.size() = " 				+ 	" 1 " 						+
                " | purpose = " 		+ predictedOutcome 	+ 	" | old.State.equals(newState) = " 	+ 	oldState.equals(newState) 	+
                " | OUTCOME = DOUBLE }");
          outcome=AggregationOutcome.DOUBLE;
          ack=true;
        }

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 1           | 1. Peers have communicated before (start time = Big Bang)               |
+------------------+-------------------------------------------------------------------------+
| line 3           | 2. Only a single state in the list of all states that the sending peers |
|                  |    think that we have                                                   |
+------------------+-------------------------------------------------------------------------+
| line 9           | 3. Other peer thinks we should EXPLOIT it                               |
|                  |    (recall that there is only 1 state in posAMS)                        |
+------------------+-------------------------------------------------------------------------+
| line 11          | posAMS[0] != new selected state                                         |
|                  | (in case 6, this is not true -> old = new)                              |
+------------------+-------------------------------------------------------------------------+
| line 14 - 32     | Do nothing                                                              |
+------------------+-------------------------------------------------------------------------+
| line 29          | Successfully replaced (since we had nothing to do)                      |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:1
  | positiveDMA:1
  | posAMS.size:1
  | purpose:EXPLOITATION*
  | posAMS[0] != SELECTED_STATE


**Notes**

  | * shouldRemove:false
  | newState = DisseminatorReport.SELECTED_STATE


Case 7
------

.. .. image:: aggregation.mechanism/case.7.png

.. **Figure** DIAS Aggregation Case 7

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    if(posAMS.size() == 1)
    {
      // Not executed in case 7, omitted here
    }
    else
    {
      if(posAMS.size() == 0 && !shouldRemove)
      {
        AMSAddition = newState;
        AMSRemoval = null;
        boolean bool = this.SMA.contains(newState.getStateId().toString());
        if(bool) {
          double count0 = (Double)this.aggregates.getAggregate(AggregationFunction.COUNT);

          outcome = AggregationOutcome.DOUBLE;

          logEvent( "receiveDisseminatorReport", "case", "7" );
          this.log(1,"7. CURRENT STATE { " +
                " positiveAMD = " 		+ 	positiveAMD 		+ 	" | positiveDMA = " 			+ 	positiveDMA +
                " | shouldRemove = " 		+ 	shouldRemove 		+ 	" | posAMS.size() = " 			+ 	" 0 " 		+
                " | purpose = " 			+ 	predictedOutcome 	+ 	" | SMA.contains(newState) = " 	+ 	bool 		+
                " | count0 = " 				+ 	count0 				+
                " | OUTCOME = DUPLICATE }");

          AMSAddition = newState;
          AMSRemoval = null;
          if(!tobeRemoved.isEmpty()) {
            AMSRemoval = tobeRemoved.get(0);
          }
          ack = true;

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 1           | 1. Peers have communicated before (start time = Big Bang)               |
+------------------+-------------------------------------------------------------------------+
| line 9           | 2. No entries in the list of states that the sending peer think that we |
|                  |    have                                                                 |
+------------------+-------------------------------------------------------------------------+
| line 9           | 3. Sending peer sends an EXPLOITATION                                   |
+------------------+-------------------------------------------------------------------------+
| line 14          | New selected state is already in our aggregation                        |
+------------------+-------------------------------------------------------------------------+
| line 29 - 31     | Jovan and Edward are still unclear why this code is here                |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:1
  | positiveDMA:1
  | posAMS.size:0
  | purpose:EXPLOITATION*


**Notes**

  | * shouldRemove:false
  | newState = DisseminatorReport.SELECTED_STATE


Case 8
------

.. .. image:: aggregation.mechanism/case.8.png

.. **Figure** DIAS Aggregation Case 8

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    if(posAMS.size() == 1)
    {
      // Not executed in case 8, omitted here
    }
    else
    {
      if(posAMS.size() == 0 && !shouldRemove)
      {
        AMSAddition = newState;
        AMSRemoval = null;
        boolean bool = this.SMA.contains(newState.getStateId().toString());
        if(bool)
        {
          // Not executed in case 8, omitted here
        }
        else
        {
          double count0 = (Double)this.aggregates.getAggregate(AggregationFunction.COUNT);

          logEvent( "receiveDisseminatorReport", "case", "8" );
          this.log(1,"8. CURRENT STATE { " +
                " positiveAMD = " 	+ positiveAMD 		+ 	" | positiveDMA = " 			+ 	positiveDMA 	+
                " | shouldRemove = " 	+ shouldRemove 		+ 	" | posAMS.size() = " 			+ 	"0 " 			+
                " | purpose = " 		+ predictedOutcome 	+	" | SMA.contains(newState) = "	+	bool 			+
                " | count0 = " 				+ 	count0 				+
                " | NOT INCONSISTENCY ANYMORE }");
          if(possibles != null) {
            Iterator<State> iterator = possibles.iterator();
            boolean found = false;
            State state = null;
            while(iterator.hasNext()) {
              state = iterator.next();
              if(this.removeSMAMembership(state)) {
                this.removeAggregationState(state);
                found = true;
              }
            }
            double count1 = (Double)this.aggregates.getAggregate(AggregationFunction.COUNT);

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 1           | 1. Peers have communicated before (start time = Big Bang)               |
+------------------+-------------------------------------------------------------------------+
| line 9           | 2. No entries in the list of states that the sending peer think that we |
|                  |    have                                                                 |
+------------------+-------------------------------------------------------------------------+
| line 9           | 3. Sending peer did not send a REMOVAL                                  |
|                  |    (this is the correct way of interpreting this flag)                  |
+------------------+-------------------------------------------------------------------------+
| line 18          | 4. New selected state is NOT in our aggregation                         |
+------------------+-------------------------------------------------------------------------+
| line 29          | 5. Sending peer sent it's possible states                               |
+------------------+-------------------------------------------------------------------------+
| line 33 - 39     | **To be safe**, remove all possible states of the sending peer from our |
|                  | Aggregator                                                              |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:1
  | positiveDMA:1
  | posAMS.size:0
  | purpose:EXPLOITATION*


**Notes**

  | * shouldRemove:false
  | newState = DisseminatorReport.SELECTED_STATE
  | possibles = DisseminatorReport.ALL_POSSIBLE_STATES


Case 8.1
--------

.. .. image:: aggregation.mechanism/case.8.1.png

.. **Figure** DIAS Aggregation Case 8.1

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    if(posAMS.size() == 1)
    {
      // Not executed in case 8.1, omitted here
    }
    else
    {
      if(posAMS.size() == 0 && !shouldRemove)
      {
        AMSAddition = newState;
        AMSRemoval = null;
        boolean bool = this.SMA.contains(newState.getStateId().toString());
        if(bool)
        {
          // Not executed in case 8.1, omitted here
        }
        else
        {
          double count0 = (Double)this.aggregates.getAggregate(AggregationFunction.COUNT);

          logEvent( "receiveDisseminatorReport", "case", "8" );
          this.log(1,"8. CURRENT STATE { " +
                " positiveAMD = " 	+ positiveAMD 		+ 	" | positiveDMA = " 			+ 	positiveDMA 	+
                " | shouldRemove = " 	+ shouldRemove 		+ 	" | posAMS.size() = " 			+ 	"0 " 			+
                " | purpose = " 		+ predictedOutcome 	+	" | SMA.contains(newState) = "	+	bool 			+
                " | count0 = " 				+ 	count0 				+
                " | NOT INCONSISTENCY ANYMORE }");
          if(possibles != null) {

            //**************************************************************//
            //                                                              //
            //  code for removal of all possible states here (see Case 8)   //
            //                                                              //
            //**************************************************************//

            if(found == false)
            {
              logEvent( "receiveDisseminatorReport", "case", "8.1" );
              this.log(1,"8.1. CURRENT STATE { " +
                    " positiveAMD = " 		+ positiveAMD 		+ 	" | positiveDMA = "		+ 	positiveDMA +
                    " | shouldRemove = " 		+ shouldRemove 		+ 	" | posAMS.size() = " 	+ 	" 0 " 		+
                    " | purpose = " 			+ predictedOutcome 	+
                    " | count1 = " 				+ 	count1 				+
                    " | possibles == null ? "	+ (possibles==null)	+ 	" | NOTHING REMOVED }");
            }

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 1           | 1. Peers have communicated before (start time = Big Bang)               |
+------------------+-------------------------------------------------------------------------+
| line 9           | 2. No entries in the list of states that the sending peer think that we |
|                  |    have                                                                 |
+------------------+-------------------------------------------------------------------------+
| line 9           | 3. Sending peer sends an EXPLOITATION                                   |
+------------------+-------------------------------------------------------------------------+
| line 18          | 4. New selected state is NOT in our aggregation                         |
+------------------+-------------------------------------------------------------------------+
| line 29          | 5. Sending peer sent it's possible states                               |
+------------------+-------------------------------------------------------------------------+
| line 33          | Remove all possible states of the sending peer from our aggregator      |
+------------------+-------------------------------------------------------------------------+
| line 37          | None of the possible states were found in our aggregator                |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:1
  | positiveDMA:1
  | posAMS.size:0
  | purpose:EXPLOITATION*


**Notes**

  | * shouldRemove:false
  | newState = DisseminatorReport.SELECTED_STATE
  | possibles = DisseminatorReport.ALL_POSSIBLE_STATES

Case 8.2
--------

.. .. image:: aggregation.mechanism/case.8.2.png

.. **Figure** DIAS Aggregation Case 8.2

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    if(posAMS.size() == 1)
    {
      // Not executed in case 8.2, omitted here
    }
    else
    {
      if(posAMS.size() == 0 && !shouldRemove)
      {
        AMSAddition = newState;
        AMSRemoval = null;
        boolean bool = this.SMA.contains(newState.getStateId().toString());
        if(bool)
        {
          // Not executed in case 8.2, omitted here
        }
        else
        {
          double count0 = (Double)this.aggregates.getAggregate(AggregationFunction.COUNT);

          logEvent( "receiveDisseminatorReport", "case", "8" );
          this.log(1,"8. CURRENT STATE { " +
                " positiveAMD = " 	+ positiveAMD 		+ 	" | positiveDMA = " 			+ 	positiveDMA 	+
                " | shouldRemove = " 	+ shouldRemove 		+ 	" | posAMS.size() = " 			+ 	"0 " 			+
                " | purpose = " 		+ predictedOutcome 	+	" | SMA.contains(newState) = "	+	bool 			+
                " | count0 = " 				+ 	count0 				+
                " | NOT INCONSISTENCY ANYMORE }");
          if(possibles != null) {

            //**************************************************************//
            //                                                              //
            //  code for removal of all possible states here (see Case 8)   //
            //                                                              //
            //**************************************************************//

            if(found == false)
            {
              // Not executed int Case 8.2, omitted here
            }
            else
            {
              logEvent( "receiveDisseminatorReport", "case", "8.2" );
              this.log(1,"8.2. CURRENT STATE { " +
                    " positiveAMD = " 		+ positiveAMD 		+ 	" | positiveDMA = "		+ 	positiveDMA +
                    " | shouldRemove = " 		+ shouldRemove 		+ 	" | posAMS.size() = " 	+ 	" 0 " 		+
                    " | purpose = " 			+ predictedOutcome 	+
                    " | count1 = " 				+ 	count1 				+
                    " | possibles == null ? "	+ (possibles==null)	+ 	" | STATES REMOVED FROM SMA }");
            }

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 1           | 1. Peers have communicated before (start time = Big Bang)               |
+------------------+-------------------------------------------------------------------------+
| line 9           | 2. No entries in the list of states that the sending peer think that we |
|                  |    have                                                                 |
+------------------+-------------------------------------------------------------------------+
| line 9           | 3. Sending peer sends an EXPLOITATION                                   |
+------------------+-------------------------------------------------------------------------+
| line 18          | 4. New selected state is NOT in our aggregation                         |
+------------------+-------------------------------------------------------------------------+
| line 29          | 5. Sending peer sent it's possible states                               |
+------------------+-------------------------------------------------------------------------+
| line 33          | Remove all possible states of the sending peer from our aggregator      |
+------------------+-------------------------------------------------------------------------+
| line 41          | At least one of the possible states was found in the Aggregator         |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:1
  | positiveDMA:1
  | posAMS.size:0
  | purpose:EXPLOITATION*


**Notes**

  | * shouldRemove:false
  | newState = DisseminatorReport.SELECTED_STATE
  | possibles = DisseminatorReport.ALL_POSSIBLE_STATES


Case 8.3
--------

.. .. image:: aggregation.mechanism/case.8.3.png

.. **Figure** DIAS Aggregation Case 8.3

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    if(posAMS.size() == 1)
    {
      // Not executed in case 8.3, omitted here
    }
    else
    {
      if(posAMS.size() == 0 && !shouldRemove)
      {
        AMSAddition = newState;
        AMSRemoval = null;
        boolean bool = this.SMA.contains(newState.getStateId().toString());
        if(bool)
        {
          // Not executed in case 8.3, omitted here
        }
        else
        {
          double count0 = (Double)this.aggregates.getAggregate(AggregationFunction.COUNT);

          logEvent( "receiveDisseminatorReport", "case", "8" );
          this.log(1,"8. CURRENT STATE { " +
                " positiveAMD = " 	+ positiveAMD 		+ 	" | positiveDMA = " 			+ 	positiveDMA 	+
                " | shouldRemove = " 	+ shouldRemove 		+ 	" | posAMS.size() = " 			+ 	"0 " 			+
                " | purpose = " 		+ predictedOutcome 	+	" | SMA.contains(newState) = "	+	bool 			+
                " | count0 = " 				+ 	count0 				+
                " | NOT INCONSISTENCY ANYMORE }");
          if(possibles != null) {

            //************************************************************************//
            //                                                                        //
            //  code for removal of all possible states here (see cases 8.1 and 8.2)  //
            //                                                                        //
            //************************************************************************//

            AMSAddition = newState;
            boolean SMAbool = this.addSMAMembership(newState); // returns true if the selected state membership is added in the SMA bloom filter
            if(SMAbool)
            {
              this.addAggregationState(newState);
              outcome = AggregationOutcome.FIRST;
              ack = true;
              logEvent( "receiveDisseminatorReport", "case", "8.3" );

              double count2 = (Double)this.aggregates.getAggregate(AggregationFunction.COUNT);
              this.log(1,"8.3. CURRENT STATE { " +
                    " positiveAMD = " 		+ positiveAMD 		+ 	" | positiveDMA = "		+ 	positiveDMA +
                    " | shouldRemove = " 		+ shouldRemove 		+ 	" | posAMS.size() = " 	+ 	" 0 " 		+
                    " | purpose = " 			+ predictedOutcome 	+ 	" | addSMAMembership(newState) = "	+ SMAbool	+
                    " | count2 = " 				+ 	count2 				+
                    " | possibles == null ? "	+ (possibles==null)	+ 	" | OUTCOME = EXPLOITATION }");
            }

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 1           | 1. Peers have communicated before (start time = Big Bang)               |
+------------------+-------------------------------------------------------------------------+
| line 9           | 2. No entries in the list of states that the sending peer think that we |
|                  |    have                                                                 |
+------------------+-------------------------------------------------------------------------+
| line 9           | 3. Sending peer sends an EXPLOITATION                                   |
+------------------+-------------------------------------------------------------------------+
| line 18          | 4. New selected state is NOT in our aggregation                         |
+------------------+-------------------------------------------------------------------------+
| line 29          | 5. Sending peer sent it's possible states                               |
+------------------+-------------------------------------------------------------------------+
| line 33          | Remove all possible states of the sending peer from our aggregator      |
+------------------+-------------------------------------------------------------------------+
| line 39          | 6. True means the new selected state was not yet in our SMA             |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:1
  | positiveDMA:1
  | posAMS.size:0
  | purpose:EXPLOITATION*


**Notes**

  | * shouldRemove:false
  | newState = DisseminatorReport.SELECTED_STATE
  | possibles = DisseminatorReport.ALL_POSSIBLE_STATES


Case 8.4
--------

.. .. image:: aggregation.mechanism/case.8.4.png

.. **Figure** DIAS Aggregation Case 8.4

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    if(posAMS.size() == 1)
    {
      // Not executed in case 8.4, omitted here
    }
    else
    {
      if(posAMS.size() == 0 && !shouldRemove)
      {
        AMSAddition = newState;
        AMSRemoval = null;
        boolean bool = this.SMA.contains(newState.getStateId().toString());
        if(bool)
        {
          // Not executed in case 8.4, omitted here
        }
        else
        {
          double count0 = (Double)this.aggregates.getAggregate(AggregationFunction.COUNT);

          logEvent( "receiveDisseminatorReport", "case", "8" );
          this.log(1,"8. CURRENT STATE { " +
                " positiveAMD = " 	+ positiveAMD 		+ 	" | positiveDMA = " 			+ 	positiveDMA 	+
                " | shouldRemove = " 	+ shouldRemove 		+ 	" | posAMS.size() = " 			+ 	"0 " 			+
                " | purpose = " 		+ predictedOutcome 	+	" | SMA.contains(newState) = "	+	bool 			+
                " | count0 = " 				+ 	count0 				+
                " | NOT INCONSISTENCY ANYMORE }");
          if(possibles != null) {

            //************************************************************************//
            //                                                                        //
            //  code for removal of all possible states here (see cases 8.1 and 8.2)  //
            //                                                                        //
            //************************************************************************//

            AMSAddition = newState;
            boolean SMAbool = this.addSMAMembership(newState); // returns true if the selected state membership is added in the SMA bloom filter
            if(SMAbool)
            {
              // Not executed in case 8.4, omitted here
            }
            else
            {
              logEvent( "receiveDisseminatorReport", "case", "8.4" );
              this.log(1,"8.4. CURRENT STATE { " +
                    " positiveAMD = " 		+ positiveAMD 		+ 	" | positiveDMA = "		+ 	positiveDMA +
                    " | shouldRemove = " 		+ shouldRemove 		+ 	" | posAMS.size() = " 	+ 	" 0 " 		+
                    " | purpose = " 			+ predictedOutcome 	+ 	" | addSMAMembership(newState) = "	+	SMAbool	+
                    " | possibles == null ? "	+ (possibles==null)	+ 	" | OUTCOME = INCONSISTENCY }");
              outcome = AggregationOutcome.UNSUCCESSFUL;
              ack = false;
            }

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 1           | 1. Peers have communicated before (start time = Big Bang)               |
+------------------+-------------------------------------------------------------------------+
| line 9           | 2. No entries in the list of states that the sending peer think that we |
|                  |    have                                                                 |
+------------------+-------------------------------------------------------------------------+
| line 9           | 3. Sending peer sends an EXPLOITATION                                   |
+------------------+-------------------------------------------------------------------------+
| line 18          | 4. New selected state is NOT in our aggregation                         |
+------------------+-------------------------------------------------------------------------+
| line 29          | 5. Sending peer sent it's possible states                               |
+------------------+-------------------------------------------------------------------------+
| line 33          | Remove all possible states of the sending peer from our aggregator      |
+------------------+-------------------------------------------------------------------------+
| line 43          | 6. New selected state was already in our SMA                            |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:1
  | positiveDMA:1
  | posAMS.size:0
  | purpose:EXPLOITATION*


**Notes**

  | * shouldRemove:false
  | newState = DisseminatorReport.SELECTED_STATE
  | possibles = DisseminatorReport.ALL_POSSIBLE_STATES

Case 8.5
--------

.. .. image:: aggregation.mechanism/case.8.5.png

.. **Figure** DIAS Aggregation Case 8.5

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    if(posAMS.size() == 1)
    {
      // Not executed in case 8.5, omitted here
    }
    else
    {
      if(posAMS.size() == 0 && !shouldRemove)
      {
        AMSAddition = newState;
        AMSRemoval = null;
        boolean bool = this.SMA.contains(newState.getStateId().toString());
        if(bool)
        {
          // Not executed in case 8.5, omitted here
        }
        else
        {
          double count0 = (Double)this.aggregates.getAggregate(AggregationFunction.COUNT);

          logEvent( "receiveDisseminatorReport", "case", "8" );
          this.log(1,"8. CURRENT STATE { " +
                " positiveAMD = " 	+ positiveAMD 		+ 	" | positiveDMA = " 			+ 	positiveDMA 	+
                " | shouldRemove = " 	+ shouldRemove 		+ 	" | posAMS.size() = " 			+ 	"0 " 			+
                " | purpose = " 		+ predictedOutcome 	+	" | SMA.contains(newState) = "	+	bool 			+
                " | count0 = " 				+ 	count0 				+
                " | NOT INCONSISTENCY ANYMORE }");
          if(possibles != null) {
            Not executed in case 8.5, omitted here
          }
          else
          {
            logEvent( "receiveDisseminatorReport", "case", "8.5" );
            this.log(1,"8.5. CURRENT STATE { " +
                  " positiveAMD = " 		+ positiveAMD 		+ 	" | positiveDMA = "		+ 	positiveDMA +
                  " | shouldRemove = " 		+ shouldRemove 		+ 	" | posAMS.size() = " 	+ 	" 0 " 		+
                  " | purpose = " 			+ predictedOutcome 	+
                  " | possibles == null ? "	+ (possibles==null)	+ 	" | NOTHING REMOVED FROM SMA }");
            outcome = AggregationOutcome.UNSUCCESSFUL;
            ack = false;
          }

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 1           | 1. Peers have communicated before (start time = Big Bang)               |
+------------------+-------------------------------------------------------------------------+
| line 9           | 2. No entries in the list of states that the sending peer think that we |
|                  |    have                                                                 |
+------------------+-------------------------------------------------------------------------+
| line 9           | 3. Sending peer sends an EXPLOITATION                                   |
+------------------+-------------------------------------------------------------------------+
| line 18          | 4. New selected state is NOT in our aggregation                         |
+------------------+-------------------------------------------------------------------------+
| line 32          | 5. Sending peer did NOT send it's possible states                       |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:1
  | positiveDMA:1
  | posAMS.size:0
  | purpose:EXPLOITATION*


**Notes**

  | * shouldRemove:false
  | newState = DisseminatorReport.SELECTED_STATE
  | possibles = DisseminatorReport.ALL_POSSIBLE_STATES

Case 9
------

.. .. image:: aggregation.mechanism/case.9.png

.. **Figure** DIAS Aggregation Case 9

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    if(posAMS.size() == 1)
    {
      // Not executed in case 9, omitted here
    }
    else
    {
      if(posAMS.size() == 0 && !shouldRemove)
      {
        // Not executed in case 9, omitted here
      }
      else
      {
        if(posAMS.size() == 0 && shouldRemove)
        {
          AMSAddition = null;
          if(possibles != null)
          {
            boolean a = false;
            for(State state : possibles) {
              if(this.SMA.contains(state.getStateId().toString())) {
                this.removeSMAMembership(state);
                this.removeAggregationState(state);
                AMSRemoval = state;
                outcome = AggregationOutcome.REMOVAL;
                ack = true;
                a = true;
                this.log( "#####################");
                break;
              }
            }
            if(a == false)
            {
              logEvent( "receiveDisseminatorReport", "case", "9" );
              this.log(1,"9. CURRENT STATE { " +
                    " positiveAMD = " 	+ positiveAMD 		+ 	" | positiveDMA = " 			+ 	positiveDMA 	+
                    " | shouldRemove = " 	+ shouldRemove 		+ 	" | posAMS.size() = " 			+ 	" 0 " 			+
                    " | purpose = " 		+ predictedOutcome 	+	" | possibles == null ? "	+	(possibles==null)	+
                    " | NOTHING WAS REMOVED }");
              }
            }

+------------------+-------------------------------------------------------------------------------------+
| Line number      | Description                                                                         |
+==================+=====================================================================================+
| line 1           | 1. Peers have communicated before (start time = Big Bang)                           |
+------------------+-------------------------------------------------------------------------------------+
| line 9           | shouldRemove (REMOVAL) are only sent by migrated disseminators                      |
+------------------+-------------------------------------------------------------------------------------+
| line 15          | 2. No entries in the list of states that the sending peer think that we             |
|                  |    have                                                                             |
+------------------+-------------------------------------------------------------------------------------+
| line 15          | 3. Migrated peer (inside a carrier node) sends a REMOVAL                            |
+------------------+-------------------------------------------------------------------------------------+
| line 18          | 4. Sending peer sent it's possible states                                           |
+------------------+-------------------------------------------------------------------------------------+
| line 24          | **To be safe**, remove all possible states of the sending peer from our             |
|                  | Aggregator                                                                          |
+------------------+-------------------------------------------------------------------------------------+
| line 26          | | **To be safe**, inform the sending peer that the aggregator removed something,    |
|                  | | even if the sending peer has no entries for our Aggregator (i.e. posAMS is empty) |
+------------------+-------------------------------------------------------------------------------------+
| line 33          | 5. Nothing was removed on our side                                                  |
+------------------+-------------------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:1
  | positiveDMA:1
  | posAMS.size:0
  | purpose:Not


**Notes**

  | * shouldRemove:false
  | newState = DisseminatorReport.SELECTED_STATE
  | possibles = DisseminatorReport.ALL_POSSIBLE_STATES


Case 10
-------

.. .. image:: aggregation.mechanism/case.10.png

.. **Figure** DIAS Aggregation Case 10

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    if(posAMS.size() == 1)
    {
      // Not executed in case 10, omitted here
    }
    else
    {
      if(posAMS.size() == 0 && !shouldRemove)
      {
        // Not executed in case 10, omitted here
      }
      else
      {
        if(posAMS.size() == 0 && shouldRemove)
        {
          AMSAddition = null;
          if(possibles != null)
          {
            // Not executed in case 10, omitted here
          }
          else
          {
            logEvent( "receiveDisseminatorReport", "case", "10" );
            this.log(1,"10. CURRENT STATE { " +
                  " positiveAMD = " 	+ positiveAMD 		+ 	" | positiveDMA = " 			+ 	positiveDMA 	+
                  " | shouldRemove = " 	+ shouldRemove 		+ 	" | posAMS.size() = " 			+ 	" 0 " 			+
                  " | purpose = " 		+ predictedOutcome 	+	" | possibles == null ? "	+	(possibles==null)	+
                  " | NOTHING WAS REMOVED }");
          }

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 1           | 1. Peers have communicated before (start time = Big Bang)               |
+------------------+-------------------------------------------------------------------------+
| line 9           | shouldRemove (REMOVAL) are only sent by migrated disseminators          |
+------------------+-------------------------------------------------------------------------+
| line 15          | 2. No entries in the list of states that the sending peer think that we |
|                  |    have                                                                 |
+------------------+-------------------------------------------------------------------------+
| line 15          | 3. Migrated peer (inside a carrier node) sends a REMOVAL                |
+------------------+-------------------------------------------------------------------------+
| line 22          | 4. Sending peer did not send it's possible states                       |
+------------------+-------------------------------------------------------------------------+
| line 22          | Nothing was removed on our side                                         |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:1
  | positiveDMA:1
  | posAMS.size:0
  | purpose:Not


**Notes**

  | * shouldRemove:false
  | newState = DisseminatorReport.SELECTED_STATE
  | possibles = DisseminatorReport.ALL_POSSIBLE_STATES


Case 11
-------

.. .. image:: aggregation.mechanism/case.11.png

.. **Figure** DIAS Aggregation Case 11

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    if(posAMS.size() == 1)
    {
      // Not executed in case 11, omitted here
    }
    else
    {
      if(posAMS.size() == 0 && !shouldRemove)
      {
        // Not executed in case 11, omitted here
      }
      else
      {
        if(posAMS.size() == 0 && shouldRemove)
        {
          // Not executed in case 11, omitted here
        }
        else
        {
          logEvent( "receiveDisseminatorReport", "case", "11" );
          this.log(1,"11. CURRENT STATE { " +
                " positiveAMD = " 	+ positiveAMD 		+ 	" | positiveDMA = " 			+ 	positiveDMA 	+
                " | shouldRemove = " 	+ shouldRemove 		+ 	" | posAMS.size() = " 			+ 	posAMS.size()	+
                " | purpose = " 		+ predictedOutcome 	+
                " | OUTCOME = INCONSISTENCY_2 }");
          outcome = AggregationOutcome.UNSUCCESSFUL;
          ack = false;
        }

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 1           | 1. Peers have communicated before (start time = Big Bang)               |
+------------------+-------------------------------------------------------------------------+
| line 9           | shouldRemove (REMOVAL) are only sent by migrated disseminators          |
+------------------+-------------------------------------------------------------------------+
| line 15          | 2. No entries in the list of states that the sending peer think that we |
|                  |    have                                                                 |
+------------------+-------------------------------------------------------------------------+
| line 15          | 3. Migrated peer (inside a carrier node) sends a REMOVAL                |
+------------------+-------------------------------------------------------------------------+
| line 19          | 4. posAMS.size > 1                                                      |
+------------------+-------------------------------------------------------------------------+
| line 27          | Normally this should not happen, because posAMS should never be >= 1    |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:1
  | positiveDMA:1
  | posAMS.size: >1

Case 12
-------

.. .. image:: aggregation.mechanism/case.12.png

.. **Figure** DIAS Aggregation Case 12

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    // Not executed in case 12, omitted here
  }
  else
  {
      if(!positiveAMD && positiveDMA && shouldRemove)
      {
        AMSAddition = null;
        HashSet<State> possibles = (HashSet<State>) disseminatorReport.get(DisseminatorReport.ALL_POSSIBLE_STATES);
        if(possibles != null)
        {
          boolean a = false;
          for(State state : possibles) {
            if(this.SMA.contains(state.getStateId().toString()))
            {
              this.removeSMAMembership(state);
              this.removeAggregationState(state);
              AMSRemoval = state;
              outcome = AggregationOutcome.REMOVAL;
              a = true;
              ack = true;
              System.err.println("\t\t\t\t\t^&*^&*^&*^&*^&*^&*^&*^&*^&*^&*");
              break;
            }
          }
          if(a == false)
          {
            logEvent( "receiveDisseminatorReport", "case", "12" );
            this.log(1,"12. CURRENT STATE { " +
                  " positiveAMD = " 	+ positiveAMD 		+ 	" | positiveDMA = " 			+ 	positiveDMA 	+
                  " | shouldRemove = " 	+ shouldRemove 		+ 	" | posAMS.size() = " 			+ 	"NO POS AMS"	+
                  " | purpose = " 		+ predictedOutcome 	+	" | possibles == null ? "	+	(possibles==null)	+
                  " | NOTHING WAS REMOVED }");
          }

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 7           | | 1. Only the receiving peer believes we have communicared before       |
|                  | | (typically when the sending peer never received the PUSH message)     |
+------------------+-------------------------------------------------------------------------+
| line 7           | shouldRemove (REMOVAL) are only sent by migrated disseminators          |
+------------------+-------------------------------------------------------------------------+
| line 18          | 2. Remove all possible states of the sending peer in the aggregator     |
+------------------+-------------------------------------------------------------------------+
| line 20          | 3. Migrated peer (inside a carrier node) sends a REMOVAL                |
+------------------+-------------------------------------------------------------------------+
| line 27          | 4. Nothing got removed                                                  |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:0
  | positiveDMA:1
  | purpose: REMOVAL


Case 13
-------

.. .. image:: aggregation.mechanism/case.13.png

.. **Figure** DIAS Aggregation Case 13

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    // Not executed in case 13, omitted here
  }
  else
  {
      if(!positiveAMD && positiveDMA && shouldRemove)
      {
        AMSAddition = null;
        HashSet<State> possibles = (HashSet<State>) disseminatorReport.get(DisseminatorReport.ALL_POSSIBLE_STATES);
        if(possibles != null)
        {
          // Not executed in case 13, omitted here
        }
        else
        {
          logEvent( "receiveDisseminatorReport", "case", "13" );
          this.log(1,"13. CURRENT STATE { " +
                " positiveAMD = " 	+ positiveAMD 		+ 	" | positiveDMA = " 			+ 	positiveDMA 	+
                " | shouldRemove = " 	+ shouldRemove 		+ 	" | posAMS.size() = " 			+ 	"NO POS AMS"	+
                " | purpose = " 		+ predictedOutcome 	+	" | possibles == null ? "	+	(possibles==null)	+
                " | NOTHING WAS REMOVED }");
        }

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 7           | | 1. Only the receiving peer believes we have communicared before       |
|                  | | (typically when the sending peer never received the PUSH message)     |
+------------------+-------------------------------------------------------------------------+
| line 7           | shouldRemove (REMOVAL) are only sent by migrated disseminators          |
+------------------+-------------------------------------------------------------------------+
| line 15          | 2. Nothing gets removed                                                 |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:0
  | positiveDMA:1
  | purpose: REMOVAL

Case 14
-------

.. .. image:: aggregation.mechanism/case.14.png

.. **Figure** DIAS Aggregation Case 14

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    // Not executed in case 14, omitted here
  }
  else
  {
      if(!positiveAMD && positiveDMA && shouldRemove)
      {
        // Not executed in case 14, omitted here
      }
      else
      {
        if(!positiveAMD && positiveDMA && !shouldRemove)
        {
          if("EXPLOITATION".equals(predictedOutcome))
          {
            AMSAddition = newState;
            outcome = AggregationOutcome.DOUBLE;
            ack = true;

            logEvent( "receiveDisseminatorReport", "case", "14" );
            this.log(1,"14. CURRENT STATE { " +
                  " positiveAMD = " 	+ positiveAMD 		+ 	" | positiveDMA = " 	+ 	positiveDMA 	+
                  " | shouldRemove = " 	+ shouldRemove 		+ 	" | posAMS.size() = " 	+ 	"NO POS AMS"	+
                  " | purpose = " 		+ predictedOutcome 	+	" | ack = " 			+ 	ack				+
                  " | OUTCOME = DUPLICATE }");
          }

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 7           | | 1. Only the receiving peer believes we have communicared before       |
|                  | | (typically when the sending peer never received the PUSH message)     |
+------------------+-------------------------------------------------------------------------+
| line 7           | shouldRemove (REMOVAL) are only sent by migrated disseminators          |
+------------------+-------------------------------------------------------------------------+
| line 15          | | If the current version, where predictedOutcome can only be REMOVAL or |
|                  | | EXPLOITATION, this line of code is redundant and could be removed     |
+------------------+-------------------------------------------------------------------------+
| line 15          | Duplicate, nothing gets changed                                         |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:0
  | positiveDMA:1
  | purpose: UPDATE

Case 15
-------

.. .. image:: aggregation.mechanism/case.15.png

.. **Figure** DIAS Aggregation Case 15

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    // Not executed in case 15, omitted here
  }
  else
  {
      if(!positiveAMD && positiveDMA && shouldRemove)
      {
        // Not executed in case 15, omitted here
      }
      else
      {
        if(!positiveAMD && positiveDMA && !shouldRemove)
        {
          if("EXPLOITATION".equals(predictedOutcome))
          {
            // Not executed in case 15, omitted here
          }
          else
          {
            outcome = AggregationOutcome.DOUBLE;
            ack = false;

            logEvent( "receiveDisseminatorReport", "case", "15" );
            this.log(1,"15. CURRENT STATE { " +
                  " positiveAMD = " 	+ positiveAMD 		+ 	" | positiveDMA = " 	+ 	positiveDMA 	+
                  " | shouldRemove = " 	+ shouldRemove 		+ 	" | posAMS.size() = " 	+ 	"NO POS AMS"	+
                  " | purpose = " 		+ predictedOutcome 	+	" | ack = " 			+ 	ack				+
                  " | OUTCOME = DUPLICATE }");
          }

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 7           | shouldRemove (REMOVAL) are only sent by migrated disseminators          |
+------------------+-------------------------------------------------------------------------+
| line 15          | 1. Conditions:                                                          |
|                  |                                                                         |
|                  |    AMD = 1, DMA = 0                                                     |
|                  |                                                                         |
|                  |    AMD = 0, DMA = 0                                                     |
|                  |                                                                         |
|                  |    This can be compressed to just DMA = 0                               |
+------------------+-------------------------------------------------------------------------+
| line 19          | Duplicate, nothing gets changed                                         |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveAMD:0
  | positiveDMA:1
  | purpose: UPDATE

Case 16
-------

.. .. image:: aggregation.mechanism/case.16.png

.. **Figure** DIAS Aggregation Case 16

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    // Not executed in case 16, omitted here
  }
  else
  {
      if(!positiveAMD && positiveDMA && shouldRemove)
      {
        // Not executed in case 16, omitted here
      }
      else
      {
        if(!positiveAMD && positiveDMA && !shouldRemove)
        {
          // Not executed in case 16, omitted here
        }
        else
        {
          logEvent( "receiveDisseminatorReport", "case", "16" );
          this.log(1,"16. CURRENT STATE { " +
                " positiveAMD = " 	+ positiveAMD 		+ 	" | positiveDMA = " 	+ 	positiveDMA 	+
                " | shouldRemove = " 	+ shouldRemove 		+ 	" | posAMS.size() = " 	+ 	"NO POS AMS"	+
                " | purpose = " 		+ predictedOutcome 	+	" | ack = " 			+ 	ack				+
                " | ... }");
        }

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 7           | shouldRemove (REMOVAL) are only sent by migrated disseminators          |
+------------------+-------------------------------------------------------------------------+
| line 13          | 1. Conditions:                                                          |
|                  |                                                                         |
|                  |    AMD = 1, DMA = 0                                                     |
|                  |                                                                         |
|                  |    AMD = 0, DMA = 0                                                     |
|                  |                                                                         |
|                  |    This can be compressed to just DMA = 0                               |
+------------------+-------------------------------------------------------------------------+
| line 17 - 25     | Do nothing                                                              |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveDMA:0

Case 17
-------

.. .. image:: aggregation.mechanism/case.17.png

.. **Figure** DIAS Aggregation Case 17

.. code-block:: java
  :linenos:

  if(positiveAMD && positive DMA)
  {
    // Not executed in case 16, omitted here
  }
  else
  {
      if(!positiveAMD && positiveDMA && shouldRemove)
      {
        // Not executed in case 16, omitted here
      }
      else
      {
        if(!positiveAMD && !positiveDMA)
        {
          AMSAddition = newState;
          boolean DMAbool = this.addDMAMembership(node);
          boolean SMAbool = this.addSMAMembership(newState);

          if(DMAbool && SMAbool)
          {
            this.addAggregationState(newState);
            outcome = AggregationOutcome.FIRST;
            ack = true;

            logEvent( "receiveDisseminatorReport", "case", "17" );
            this.log(1,"17. CURRENT STATE { " +
                  " positiveAMD = " 	+ positiveAMD 		+ 	" | positiveDMA = " 	+ 	positiveDMA 	+
                  " | shouldRemove = " 	+ shouldRemove 		+ 	" | posAMS.size() = " 	+ 	"NO POS AMS"	+
                  " | purpose = " 		+ predictedOutcome 	+	" | addDMAMembership(newNode) = " + DMAbool +
                  " | addSMAMembership(newState) = " + SMAbool+	" | ack = " 			+ 	ack				+
                  " | OUTCOME = EXPLOITATION }");
          }

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 13          | 1. Peers have never communicated before                                 |
+------------------+-------------------------------------------------------------------------+
| line 21          | 2. Peer and new state successfully added to the BloomFilters            |
+------------------+-------------------------------------------------------------------------+
| line 22 - 32     | Do nothing                                                              |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 1
  | positiveDMA:0
  | positiveAMD:0

Case 20
-------

.. .. image:: aggregation.mechanism/case.20.png

.. **Figure** DIAS Aggregation Case 20

.. code-block:: java
  :linenos:

  if(this.myTopic.equals(otherTopic))
  {
    // Not executed in case 20, omitted here
  }
  else
  {
    ack = false;

    logEvent( "receiveDisseminatorReport", "case", "20" );
    this.log(1,"20. CURRENT STATE { " +
          " positiveAMD = " 	+ positiveAMD 		+ 	" | positiveDMA = " 	+ 	positiveDMA 	+
          " | shouldRemove = " 	+ shouldRemove 		+ 	" | posAMS.size() = " 	+ 	"NO POS AMS"	+
          " | purpose = " 		+ predictedOutcome 	+	" | ack = " 			+ 	ack				+
          " | OUTCOME = INCOCNSISTENCY DUE TO TOPIC MISMATCH }");
    outcome = AggregationOutcome.UNSUCCESSFUL;
    AMSAddition = null;
    AMSRemoval = null;
    aggregatorReport.put(AggregatorReport.ACK, ack);
    aggregatorReport.put(AggregatorReport.OUTCOME, outcome);
    aggregatorReport.put(AggregatorReport.AMS_ADDITION, AMSAddition);
    aggregatorReport.put(AggregatorReport.AMS_REMOVAL, AMSRemoval);
    disseminatorReport.clear();
    return aggregatorReport;
  }

+------------------+-------------------------------------------------------------------------+
| Line number      | Description                                                             |
+==================+=========================================================================+
| line 5           | 1. Topic mismatch                                                       |
+------------------+-------------------------------------------------------------------------+
| line 5 - 24      | Do nothing                                                              |
+------------------+-------------------------------------------------------------------------+

**Encoding:**

  | Topic Matched = 0
