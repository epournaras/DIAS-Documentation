Conceptual Architecture
***********************

The exact process by which DIAS performs aggregations is described below.

This process is not simple, for a number of reasons, but it is primarily due to the use of Bloom Filters. Because Bloom filters can generate false positives by their very design, the use of Bloom Filters in a DIAS network creates an uncertainty since it is no longer possible to keep track of communication history with 100% certainty. In order to decrease the probability of incorrect decisions, additional checks are made to reduce noise and therefore increase the precision of the aggregation.

The second reason is due to information latency in large networks. Uncertainty exists because information does not spread instantaneously throughout the network, which again generates a source of noise in the aggregation.

The exact process by which a peer uses all available information that it has collected in order to update it's aggregate estimates is described below.


