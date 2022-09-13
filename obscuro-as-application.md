# Editing Diagrams
Diagrams may be edited using an online PlantUML editor, such as [https://plantuml.com/](https://plantuml.com/).

# Basic Interaction

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

participant Application as A
participant Wallet as W
participant WalletExtension as WE
participant ObscuroNode as O
participant Ethereum as E

note over O, E: supports Ethereum JSON-RPC
A -> W: raw transaction
W -> W: sign
W -> WE: encrypt
WE -> O: dispatch
O -> E: fetch code and state
O -> O: process state, encrypt
O -> E: submit new state
E -> O: receipt
O -> WE: encrypted receipt
WE -> W: receipt

@enduml
```

# More Complex Example Of Guessing Game

[Image of interaction](https://www.plantuml.com/plantuml/png/bPBDRjim48JlV8fjUac19cY2zcB0HHoN4504KP0Oq2KNLbhRX2Kkkrmb_K7UlQOjsMAh77eKlvaTpb9NGGnBszJgZMUpQWm8qYyGRb65ZNNUi6cW8KVbcgb1M9ew315JwwgIs273nQS126jJqRDrgtyi0R-tw4hyhG1cpFGyfveOtkhvnPVBZzl3EyDYI-kDasjJRbQxZBseM5l1loJ45NAARqamjQPiwBckjy9ubLA8NmUlJBknIxonRcJYow2o8zdL7Hi_Flb5AN_i23FlQriQiUJ019Wbi31rZQ9_2BhG2Of4a7yBPSjqInL6c2VmHCiQ6TlcqKN1ILJeHA7lvvSVul4YDMOjXc3Twj5bfjaRCwLYLw0dPCZVWvv0QBqdbW1z3dnzo6_Fxk_cIasd2zgWY_MOdamzdQePd7s6Oh8FFJSxWATPzauLaUGJG5VoJozOFReXpYFM8yRzZMfSIWV1XlWYnq5AH-zY0aENxk8OIxHUzlEBGVo1597CZihOgz_De55_59TrleeVfoEbe2TzdzI6edlbWnZ1qq6zL1jR7Xl6Fyp3GckLTuaLHjRroYQ7tApRXR1nOOcS9lIdO9qWbxWRdXrM5JovW2fFMobqrNYVCD8dVSZ1DVxZ1QwsYMrJ_m40)

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

participant Application as A
participant WalletExtension as W
participant ObscuroNode as O
participant "Ethereum\nNode" as E
participant ERC20 as T
participant "Guessing\nGame" as G

note over O, E: supports Ethereum JSON-RPC

==Token Approval==
A -> W: ERC20.approve(GG)
W -> O: encrypted\napprove(GG)
O -> E: ERC20.approve(GG)
note left: ERC20 is not encrypted
E -> T: approve(GG)
E -> O: receipt
O -> W: encrypted receipt
W -> A: receipt


==Guessing==
A -> W: GuessingGame.guess()
W -> O: encrypted guess()
O -> E: GuessingGame.getEncryptedState()
note left: GuessingGame is encrypted
group TEE
O -> O: decrypt
O -> G: guess()
G -> T: transfer()
note left: transfer intercepted
G -> O: response
O -> O: encrypt
end
O -> E: GuessingGame.setEncryptedState()
O -> E: ERC20.transfer()
O -> W: encrypted receipt
W -> A: receipt

@enduml
```

The problem with this mixed model, where two contracts interact and one is outside the encrypted context, is that all contracts need to interact while
based on a single world view, and all transactions need to be committed atomically. In the case of the Guessing Game, the guess() interaction and the 
transfer() interaction must both happen or none happen. The only way this can happen is for the block producer to execute a transaction chain, which means 
that the Obscuro node (which is not a block producer) would need to gather the world view of the Game and ERC20 states, process the transaction logic,
rewrite the resulting transactions in such a way that a non-SGX block producer could understand, and then submit a single transaction into the network 
mempool.

