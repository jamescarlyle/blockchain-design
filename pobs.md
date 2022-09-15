# PoBI too - Proof of Batch Size (PoBS)

PoBS is a decentralised protocol for Layer 2s that ensures instant soft-finality and very low transaction costs.


## High-level overview

With a traditional approach, the finality of an L2 can always be at most as good as the speed of the L1 (for ethereum: 12 seconds * at least 5 confirmations), 
and can be achieved by publishing a rollup every block.  
This turns out to be prohibitively expensive for users even during reasonably high usage of the network.
The traditional approach is that the winning miner is decided based on a produced block, basically after a round has finished.

L2 networks that have a centralised sequencer are able to achieve instant soft-finality because there is a party that is agreed upon
ahead of time which has the authority to decide the ordering.

*Our main insight, is that the key to instant soft-finality is the fact that the sequencer is assigned ahead of time.*

We can design a decentralised protocol around this, with a strongly incentivised sequencer that is fairly assigned before each round. 
This combines the benefits of having a single sequencer with the decentralisation advantages.
If the leader is unambiguous and baked into the protocol, then transaction receipts issued by them can be used by end-users as guarantees of soft-finality.

Transactions can be very cheap if the rounds are relatively long and rollups are published infrequently to the L1, which reduces the computational overhead.
It also requires that the system uses a cheap data availability layer, until Ethereum implements the danksharding changes.


## Problems

1. Ensure leader election is fair and unambiguous. There can't be multiple nodes from a cluster claiming leadership.
2. The canonical L2 chain contains only rollups published by the legitimate leader.
3. There is a way for anyone (with an attested enclave) to challenge a published rollup so that the L1 bridge pauses withdrawals and protects users' funds.
4. Design a very strong incentive mechanism for the elected leader to follow the protocol:
  a) leader becoming non-responsive (stops responding with receipts, or doesn't publish the rollup, or doesn't share other useful info during the round) 
  b) leader that hacks into the secure enclave.
  c) prevent the leader from running multiple instances in parallel, and giving users invalid receipts (even though the aggregator follows the protocol). (Idea - users can use the invalid receipt as a challenge, or watcher can probe.)
  d) the software has to be highly-available, reliable and have resistance to DDoS attacks which can be very costly. 
5. Mechanism to prevent front running and MEV. (In PoBI we had a rollup delay)
6. How to achieve the fine-grained revelation period? (Without something very clever, the lowest revelation unit will be an L2 interval - which could be 6hrs) 
7. Ensure the ultimate Source of Truth is always the L1. Transaction receipts have to be interpreted based on the L1 first and on the current leader second.
8. Come up with a way to publish the data to a "Data Availability" layer, maybe as a series of smaller batches, each with a proof that can be verified by a Management contract in Ethereum. 
9. Ensure there is an unambiguous condition for finalizing a rollup and ending a round. A number of transactions or a timeout.
10. Leader switch
11. Gas auction

## Protocol

A round has 3 stages:
1. Leader Election
2. Transaction execution
3. Publishing


### 1. Leader Election

The goal is for the majority of Aggregators to agree on who is the leader of the next round. And very importantly, for the leader to 
be unambiguously known at the protocol level so that all wallet extensions and dApps can assess whether a transaction receipt is valid.

*Note: This is different from Pobi, where the leader was decided after the round, and it didn't matter if there was no agreement, 
because what mattered at the end of the day was what was publishd in the L1.*

#### Proposal

All aggregators who want to participate in the next round submit a special transaction to the current leader (for which they get a receipt).
This special transaction is generated inside their enclave and will contain a nonce. The leader will discard if multiple are received from the same aggregator, and just use the first.
Based on these transactions, it will publish in the header of the rollup who the leader of the next round is.
This information will be found in the L1, and can be used as the source of truth, and by the Management contract to enact slashing of stake in case of misbehaviour.

##### Attacks:

###### Current leader censors transactions.

The registering txs are encrypted and might arrive through gossip, so the host cannot know. 

###### The host might randomly remove transactions and see if the winner changes. 

This requires a restart of the enclave. This is solved with the delay we introduce on restarts.

###### There are multiple headers in the L1

- canonical chain

##### Other options considered
- deterministic : not good , not alive
- round of  election : - complicated


### 2. Transaction execution

This is the main stage of the protocol.
After publishing a rollup header to the L1, all nodes and wallet extensions will know who the leader of the next round is.
Everyone will submit all received transactions to the leader, who will start executing them.

Problems to consider for this round:

a) a long round means there are lots of transactions to execute. If the rollup is first seen by the new leader when it's published
then the transition will be very slow

Solution: publish small chunks to the da layer, so that everyone can keep up. Aggregate these chunks in an Mtree that will be added to the header.

why should the leader do it? 
how can someone take over the existing chunks?



- receipt has to be signed by the leader
