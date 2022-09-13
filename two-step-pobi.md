# Two Step POBI

The idea is to modify the Obscuro protocol and submit rollups to an L1 less frequently than the L2 blocks themselves are produced. 
![Two Step POBI](https://www.plantuml.com/plantuml/svg/tLPDR-D63BthLx0fq4iJ0Swsbm0frdPJe40IB8eKlUGorj3I82DI7OxMS8lsjrSZZ-tnXxNUfirXSt4WFdxmaNpIU_7A-RQssLUDKsMjm2F_0TwrcCcBeKOvLKFDn4Nbk4QOAokfoN5Bkc17w63qVMAGIcbU62eFN0H_Rv4A_3C363l7YobRTl3rtNJqu_Lt6S07vRmfJAF8JrJnKlOvLg3VAkDn3y9EezirFxB7Kwx1blfCR0SCfHIUtMd3YMZtPextXev4_2jfcTZtxHuBVpHovvqrsnxXuivNwB2juH7zWjtB1IY1-z4kQ5oM3albsS6u1wZDW_59v6GFEJc9dEuXfvaArC2lWawoB3G9_15TLzTbWhg190QnHe6w5H-soeH7liBE0po3He-kDeImg91MXklLYWdCi3IKmyyQbRNBO5p2fbAFO6WTgMGHq-HPrlr_TVL3oiBDEcls3HnO9uFMwTQQMUO6Vc5hs_3JaeQ7BBkzXG-L4eJE4vfsPerKOVz3LMw5RHphja2zBhPn-D5mAwiUuVPsYCG8wdcGBKCBGnJI92J7v2XfRizTo3n9aD1okTldJyQ9lmpEILI9l2Onun1YKySwKgHtfigRaj61aJnBYe3zOSHyCTT3DvIJwEaXEZ7kOiV3aQUVhECi3YTAK0DJIXSn5NW9g8eg7PSXZGsIHl9skQLBzldYUGmMc0tadUenxekHw3d4tFWqSdB0H8hS7ztzw6QzUEj-3VM7xHI9AhnXEh7UeOYdT2_0eJHCOcOMOSxk-3bTTWIPee8IlO2XmhPwTLegf9rPBbv0pNqO4sYqQfadHteDL-io2uSgL3TqVa6MBKQ5YIeMT1YOmW4KA1ABsroVtfGPXIozfUWJ7YVy24Wxguttg66sX3O8QzvHOmJCwfXhPGYQq45SXn39Sy4spuPctFU8GBZOxF6tesTgs93l2-VpDk1puWnjv_bnejfEmZYZs3x-DoeML-GjH2ocUZnFmo9uKC86uau_LSHd2rZq_BBqwznxYQozzbnYcDTUot2OhFyME-XcJsMwR4TmapDnqqDSDZWUhNibst5v9i_RNiA99JemJEfh0nxI-oy2PdcUhzyvuB10qtYOEwx1Ae_YjsyWpo1BAYh7P2JSWg_2omN-Xj4r21PCMded1rtCOIf7ELGiFWYK87c11tGljdy7XEDYUBgxUwRhFFn2YOGk50DEaUOQgArdw9xftSekgAniEXmc_SrI7uxdVqwT8UdjLy4xk5gft3gEOKf77xmDbODV2ayRZSCmMYKW7AB8mQjeNpWBex43APzyv2IV4_7v3CluoRyZg0ibeBH67HXHS34tfEp5PW_px7uKFzCsUxp5z-0DzertyDpBVuFSRxk5gPxSTd_zBhv7qcrjity0)
```
@startuml
!pragma teoz true
skinparam monochrome false
skinparam roundcorner 15
skinparam shadowing false
skinparam sequence{
  ArrowColor #EC1D24
  ParticipantBackgroundColor White
  ParticipantBorderColor White
  NoteBackgroundColor White
  NoteBorderColor Black
  ActorBorderColor Black
  ActorBackgroundColor White
  LifeLineBorderColor Black
}
skinparam note{
  BorderColor Black
  BackgroundColor White
}

participant "Ethereum Network" as L1
participant "Aggregator A" as aggregatorA
participant "Aggregator B" as aggregatorB
participant "Aggregator C" as aggregatorC
actor Users

note over L1,aggregatorC: Aggregator nodes must monitor the L1 to determine when the L2 rounds begin. Ideally they participate in the L1 gossip.

L1 --> aggregatorA: monitor
& L1 --> aggregatorB: monitor
& L1 --> aggregatorC: monitor

loop Rollup Round M

== Phase 1 - publishing the rollup produced in the previous round ==
note over L1,aggregatorC: A round begins when the winning Aggregator publishes the rollup to L1.
note over aggregatorA, aggregatorC: First, the Aggregators gossip the rollup they produced the previous round and determine who the winner is.
aggregatorA -> aggregatorB: gossip rollup M
& aggregatorA -> aggregatorC: gossip rollup M
aggregatorC -> aggregatorA: gossip rollup M
& aggregatorC -> aggregatorB: gossip rollup M

note over aggregatorA, aggregatorC: Based on the rollup nonce, each Aggregator independently determines who the winner of the round is.

aggregatorC -> aggregatorC: A is winner
& aggregatorA -> aggregatorA: A is winner
& aggregatorB -> aggregatorB: A is winner

aggregatorA -> L1: publish rollup in L1 transaction
note over L1,aggregatorA: The winner is responsible for publishing the rollup, which gets included in a L1 block after a delay.

== Phase 2 - rollup creation==
note over L1,aggregatorC: While the L1 nodes work on processing the published rollup, the L2 Aggregators process L2 transactions submitted by users. This is the main phase of the protocol.
aggregatorA -> aggregatorA: create new rollup M+1\npointing to winner
& aggregatorB -> aggregatorB: create new rollup M+1\npointing to winner
& aggregatorC -> aggregatorC: create new rollup M+1\npointing to winner


loop Block Round N
note over aggregatorA, aggregatorC: First, the Aggregators gossip the block they produced the previous round and determine who the winner is.
aggregatorA -> aggregatorB: gossip block N
& aggregatorA -> aggregatorC: gossip block N
aggregatorC -> aggregatorA: gossip block N
& aggregatorC -> aggregatorB: gossip block N

note over aggregatorA, aggregatorC: Based on the block nonce, each Aggregator independently determines who the winner of the round is.

aggregatorC -> aggregatorC: A is winner
& aggregatorA -> aggregatorA: A is winner
& aggregatorB -> aggregatorB: A is winner


aggregatorA -> aggregatorA: create new block N+1\npointing to winner
& aggregatorB -> aggregatorB: create new block N+1\npointing to winner
& aggregatorC -> aggregatorC: create new block N+1\npointing to winner

Users -> aggregatorA: L2 transactions
& Users -> aggregatorB: L2 transactions
& Users -> aggregatorC: L2 transactions

aggregatorA -> aggregatorA: Add user transactions\nto block N+1 and rollup M+1
& aggregatorB -> aggregatorB: Add user transactions\nto block N+1 and rollup M+1
& aggregatorC -> aggregatorC: Add user transactions\nto block N+1 and rollup M+1

... include receipt from latest L1 block to synchronise with L1, ~ 12 seconds later ...

aggregatorA -> aggregatorA: 1. host presents Merkle proof to TEE\n2. TEE generates random number\n3. TEE seals block N+1
& aggregatorB -> aggregatorB: 1. host presents Merkle proof to TEE\n2. TEE generates random number\n3. TEE seals block N+1
& aggregatorC -> aggregatorC: 1. host presents Merkle proof to TEE\n2. TEE generates random number\n3. TEE seals block N+1

end
== Phase 3 - nonce generation and rollup sealing==
note over L1,aggregatorC: The round ends as soon as the Aggregators independently decide that the rollup published at the beginning of the round was added to a "final" L1 block.
L1 -> L1: rollup M added to final L1 block
aggregatorA -> aggregatorA: 1. host presents Merkle proof to TEE\n2. TEE generates random number\n3. TEE seals rollup M+1
& aggregatorB -> aggregatorB: 1. host presents Merkle proof to TEE\n2. TEE generates random number\n3. TEE seals rollup M+1
& aggregatorC -> aggregatorC: 1. host presents Merkle proof to TEE\n2. TEE generates random number\n3. TEE seals rollup M+1

====
end
@enduml
```
