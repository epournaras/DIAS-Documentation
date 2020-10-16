******************
Changing of states
******************

A peer has a couple of possible states, which are values, which the connected device can possibly select as a current value.
The current value is then the selected state of the peer.
Both the possible state, as well as the selected states, can be changed over the runtime of the peer.

Changing of possible states
***************************

Possible states are a list of all states possible for the device so select/send to this specific peer.
When a selected state is chosen, it has to be a state of the possible states.
In order to keep the selected stated rather close to the actual value, the possible can/need to be updated sometimes.

In order to do that, the device sends the DIAS node a PossibleStatesMessage.

The Node then sets the list of the newly received Possible states as its own.

**Note:** If the newly received possible states are the same as the old ones, nothing gets done.


Changing of selected states
***************************

The selected state is a selected element of the elements in the possible states.

Upon receiving a new selected state through a SelectedStateMessage, the internal selected state gets changed.

**Note:** The selected state only gets changed, if the newly received one is a valid element of the internally stored possible states.
