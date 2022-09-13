# Editing Diagrams
Diagrams may be edited using an online PlantUML editor, such as [https://plantuml.com/](https://plantuml.com/).

# Basic Interaction
     ┌───────────┐          ┌──────┐          ┌───────────────┐          ┌───────────┐                ┌────────┐  
     │Application│          │Wallet│          │WalletExtension│          │ObscuroNode│                │Ethereum│  
     └─────┬─────┘          └──┬───┘          └───────┬───────┘          └─────┬─────┘                └───┬────┘  
           │                   │                      │                  ╔═════╧══════════════════════════╧══════╗
           │                   │                      │                  ║supports Ethereum JSON-RPC            ░║
           │                   │                      │                  ╚═════╤══════════════════════════╤══════╝
           │ raw transaction   │                      │                        │                          │       
           │──────────────────>│                      │                        │                          │       
           │                   │                      │                        │                          │       
           │                   │────┐                 │                        │                          │       
           │                   │    │ sign            │                        │                          │       
           │                   │<───┘                 │                        │                          │       
           │                   │                      │                        │                          │       
           │                   │       encrypt        │                        │                          │       
           │                   │ ────────────────────>│                        │                          │       
           │                   │                      │                        │                          │       
           │                   │                      │       dispatch         │                          │       
           │                   │                      │───────────────────────>│                          │       
           │                   │                      │                        │                          │       
           │                   │                      │                        │   fetch code and state   │       
           │                   │                      │                        │──────────────────────────>       
           │                   │                      │                        │                          │       
           │                   │                      │                        ────┐                              
           │                   │                      │                            │ process state, encrypt       
           │                   │                      │                        <───┘                              
           │                   │                      │                        │                          │       
           │                   │                      │                        │     submit new state     │       
           │                   │                      │                        │──────────────────────────>       
           │                   │                      │                        │                          │       
           │                   │                      │                        │         receipt          │       
           │                   │                      │                        │<──────────────────────────       
           │                   │                      │                        │                          │       
           │                   │                      │   encrypted receipt    │                          │       
           │                   │                      │<───────────────────────│                          │       
           │                   │                      │                        │                          │       
           │                   │       receipt        │                        │                          │       
           │                   │ <────────────────────│                        │                          │       
     ┌─────┴─────┐          ┌──┴───┐          ┌───────┴───────┐          ┌─────┴─────┐                ┌───┴────┐  
     │Application│          │Wallet│          │WalletExtension│          │ObscuroNode│                │Ethereum│  
     └───────────┘          └──────┘          └───────────────┘          └───────────┘                └────────┘  
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
                                                                                          ┌────────┐                           ┌────────┐                    
     ┌───────────┐          ┌───────────────┐          ┌───────────┐                      │Ethereum│          ┌─────┐          │Guessing│                    
     │Application│          │WalletExtension│          │ObscuroNode│                      │Node    │          │ERC20│          │Game    │                    
     └─────┬─────┘          └───────┬───────┘          └─────┬─────┘                      └───┬────┘          └──┬──┘          └───┬────┘                    
           │                        │                  ╔═════╧════════════════════════════════╧══════╗           │                 │                         
           │                        │                  ║supports Ethereum JSON-RPC                  ░║           │                 │                         
           │                        │                  ╚═════╤════════════════════════════════╤══════╝           │                 │                         
           │                        │                        │                                │                  │                 │                         
           │                        │                        │        ╔═══════════════╗       │                  │                 │                         
═══════════╪════════════════════════╪════════════════════════╪════════╣ Token Approval╠═══════╪══════════════════╪═════════════════╪═════════════════════════
           │                        │                        │        ╚═══════════════╝       │                  │                 │                         
           │                        │                        │                                │                  │                 │                         
           │   ERC20.approve(GG)    │                        │                                │                  │                 │                         
           │───────────────────────>│                        │                                │                  │                 │                         
           │                        │                        │                                │                  │                 │                         
           │                        │      encrypted         │                                │                  │                 │                         
           │                        │      approve(GG)       │                                │                  │                 │                         
           │                        │───────────────────────>│                                │                  │                 │                         
           │                        │                        │                                │                  │                 │                         
           │                      ╔═╧══════════════════════╗ │       ERC20.approve(GG)        │                  │                 │                         
           │                      ║ERC20 is not encrypted ░║ │────────────────────────────────>                  │                 │                         
           │                      ╚═╤══════════════════════╝ │                                │                  │                 │                         
           │                        │                        │                                │   approve(GG)    │                 │                         
           │                        │                        │                                │ ────────────────>│                 │                         
           │                        │                        │                                │                  │                 │                         
           │                        │                        │            receipt             │                  │                 │                         
           │                        │                        │<────────────────────────────────                  │                 │                         
           │                        │                        │                                │                  │                 │                         
           │                        │   encrypted receipt    │                                │                  │                 │                         
           │                        │<───────────────────────│                                │                  │                 │                         
           │                        │                        │                                │                  │                 │                         
           │        receipt         │                        │                                │                  │                 │                         
           │<───────────────────────│                        │                                │                  │                 │                         
           │                        │                        │                                │                  │                 │                         
           │                        │                        │                                │                  │                 │                         
           │                        │                        │           ╔═════════╗          │                  │                 │                         
═══════════╪════════════════════════╪════════════════════════╪═══════════╣ Guessing╠══════════╪══════════════════╪═════════════════╪═════════════════════════
           │                        │                        │           ╚═════════╝          │                  │                 │                         
           │                        │                        │                                │                  │                 │                         
           │ GuessingGame.guess()   │                        │                                │                  │                 │                         
           │───────────────────────>│                        │                                │                  │                 │                         
           │                        │                        │                                │                  │                 │                         
           │                        │   encrypted guess()    │                                │                  │                 │                         
           │                        │───────────────────────>│                                │                  │                 │                         
           │                        │                        │                                │                  │                 │                         
           │                   ╔════╧══════════════════════╗ │GuessingGame.getEncryptedState()│                  │                 │                         
           │                   ║GuessingGame is encrypted ░║ │────────────────────────────────>                  │                 │                         
           │                   ╚════╤══════════════════════╝ │                                │                  │                 │                         
           │                        │                        │                                │                  │                 │                         
           │                        │        ╔══════╤════════╪════════════════════════════════╪══════════════════╪═════════════════╪══════════════╗          
           │                        │        ║ TEE  │        │                                │                  │                 │              ║          
           │                        │        ╟──────┘        ────┐                            │                  │                 │              ║          
           │                        │        ║                   │ decrypt                    │                  │                 │              ║          
           │                        │        ║               <───┘                            │                  │                 │              ║          
           │                        │        ║               │                                │                  │                 │              ║          
           │                        │        ║               │                               guess()             │                 │              ║          
           │                        │        ║               │─────────────────────────────────────────────────────────────────────>              ║          
           │                        │        ║               │                                │                  │                 │              ║          
           │                        │        ║               │                           ╔════╧═════════════════╗│    transfer()   │              ║          
           │                        │        ║               │                           ║transfer intercepted ░║│ <────────────────              ║          
           │                        │        ║               │                           ╚════╤═════════════════╝│                 │              ║          
           │                        │        ║               │                              response             │                 │              ║          
           │                        │        ║               │<─────────────────────────────────────────────────────────────────────              ║          
           │                        │        ║               │                                │                  │                 │              ║          
           │                        │        ║               ────┐                            │                  │                 │              ║          
           │                        │        ║                   │ encrypt                    │                  │                 │              ║          
           │                        │        ║               <───┘                            │                  │                 │              ║          
           │                        │        ╚═══════════════╪════════════════════════════════╪══════════════════╪═════════════════╪══════════════╝          
           │                        │                        │                                │                  │                 │                         
           │                        │                        │GuessingGame.setEncryptedState()│                  │                 │                         
           │                        │                        │────────────────────────────────>                  │                 │                         
           │                        │                        │                                │                  │                 │                         
           │                        │                        │        ERC20.transfer()        │                  │                 │                         
           │                        │                        │────────────────────────────────>                  │                 │                         
           │                        │                        │                                │                  │                 │                         
           │                        │   encrypted receipt    │                                │                  │                 │                         
           │                        │<───────────────────────│                                │                  │                 │                         
           │                        │                        │                                │                  │                 │                         
           │        receipt         │                        │                                │                  │                 │                         
           │<───────────────────────│                        │                                │                  │                 │                         
     ┌─────┴─────┐          ┌───────┴───────┐          ┌─────┴─────┐                      ┌───┴────┐          ┌──┴──┐          ┌───┴────┐                    
     │Application│          │WalletExtension│          │ObscuroNode│                      │Ethereum│          │ERC20│          │Guessing│                    
     └───────────┘          └───────────────┘          └───────────┘                      │Node    │          └─────┘          │Game    │                    
                                                                                          └────────┘                           └────────┘            
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
rewrite the resulting transactions in such a way that a non-TEE block producer could understand, and then submit a single transaction into the network 
mempool.

# Moving World State Into TEE
                                                                                                            ┌────────┐                           ┌────────┐                    
     ┌───────────┐          ┌───────────────┐          ┌───────────┐                                        │Ethereum│          ┌─────┐          │Guessing│                    
     │Application│          │WalletExtension│          │ObscuroNode│                                        │Node    │          │ERC20│          │Game    │                    
     └─────┬─────┘          └───────┬───────┘          └─────┬─────┘                                        └───┬────┘          └──┬──┘          └───┬────┘                    
           │                        │                  ╔═════╧══════════════════════════════════════════════════╧══════╗           │                 │                         
           │                        │                  ║supports Ethereum JSON-RPC                                    ░║           │                 │                         
           │                        │                  ╚═════╤══════════════════════════════════════════════════╤══════╝           │                 │                         
           │                        │                        │                                                  │                  │                 │                         
           │                        │                        │                 ╔═══════════════╗                │                  │                 │                         
═══════════╪════════════════════════╪════════════════════════╪═════════════════╣ Token Approval╠════════════════╪══════════════════╪═════════════════╪═════════════════════════
           │                        │                        │                 ╚═══════════════╝                │                  │                 │                         
           │                        │                        │                                                  │                  │                 │                         
           │   ERC20.approve(GG)    │                        │                                                  │                  │                 │                         
           │───────────────────────>│                        │                                                  │                  │                 │                         
           │                        │                        │                                                  │                  │                 │                         
           │                        │   encrypted            │                                                  │                  │                 │                         
           │                        │   ERC20.approve(GG)    │                                                  │                  │                 │                         
           │                        │───────────────────────>│                                                  │                  │                 │                         
           │                        │                        │                                                  │                  │                 │                         
           │                        │                        │       Controller.getEncryptedStates(ERC20)       │                  │                 │                         
           │                        │                        │──────────────────────────────────────────────────>                  │                 │                         
           │                        │                        │                                                  │                  │                 │                         
           │                        │                        │                                                  │                  │                 │                         
           │                        │        ╔══════╤════════╪══════════════════════════════════════════════════╪══════════════════╪════════════╗    │                         
           │                        │        ║ TEE  │        │                                                  │                  │            ║    │                         
           │                        │        ╟──────┘        ────┐                                              │                  │            ║    │                         
           │                        │        ║                   │ decrypt                                      │                  │            ║    │                         
           │                        │        ║               <───┘                                              │                  │            ║    │                         
           │                        │        ║               │                                                  │                  │            ║    │                         
           │                        │        ║               │                             approve()            │                  │            ║    │                         
           │                        │        ║               │────────────────────────────────────────────────────────────────────>│            ║    │                         
           │                        │        ║               │                                                  │                  │            ║    │                         
           │                        │        ║               ────┐                                              │                  │            ║    │                         
           │                        │        ║                   │ encrypt                                      │                  │            ║    │                         
           │                        │        ║               <───┘                                              │                  │            ║    │                         
           │                        │        ╚═══════════════╪══════════════════════════════════════════════════╪══════════════════╪════════════╝    │                         
           │                        │                        │                                                  │                  │                 │                         
           │                        │                        │       Controller.setEncryptedStates(ERC20)       │                  │                 │                         
           │                        │                        │──────────────────────────────────────────────────>                  │                 │                         
           │                        │                        │                                                  │                  │                 │                         
           │                        │   encrypted receipt    │                                                  │                  │                 │                         
           │                        │<───────────────────────│                                                  │                  │                 │                         
           │                        │                        │                                                  │                  │                 │                         
           │        receipt         │                        │                                                  │                  │                 │                         
           │<───────────────────────│                        │                                                  │                  │                 │                         
           │                        │                        │                                                  │                  │                 │                         
           │                        │                        │                                                  │                  │                 │                         
           │                        │                        │                    ╔═════════╗                   │                  │                 │                         
═══════════╪════════════════════════╪════════════════════════╪════════════════════╣ Guessing╠═══════════════════╪══════════════════╪═════════════════╪═════════════════════════
           │                        │                        │                    ╚═════════╝                   │                  │                 │                         
           │                        │                        │                                                  │                  │                 │                         
           │ GuessingGame.guess()   │                        │                                                  │                  │                 │                         
           │───────────────────────>│                        │                                                  │                  │                 │                         
           │                        │                        │                                                  │                  │                 │                         
           │                        │ encrypted              │                                                  │                  │                 │                         
           │                        │ GuessingGame.guess()   │                                                  │                  │                 │                         
           │                        │───────────────────────>│                                                  │                  │                 │                         
           │                        │                        │                                                  │                  │                 │                         
           │                        │                        │            ERC20.getEncryptedState()             │                  │                 │                         
           │                        │                        │──────────────────────────────────────────────────>                  │                 │                         
           │                        │                        │                                                  │                  │                 │                         
           │                        │                        │Controller.getEncryptedStates(GuessingGame, ERC20)│                  │                 │                         
           │                        │                        │──────────────────────────────────────────────────>                  │                 │                         
           │                        │                        │                                                  │                  │                 │                         
           │                        │                        │                                                  │                  │                 │                         
           │                        │        ╔══════╤════════╪══════════════════════════════════════════════════╪══════════════════╪═════════════════╪══════════════╗          
           │                        │        ║ TEE  │        │                                                  │                  │                 │              ║          
           │                        │        ╟──────┘        ────┐                                              │                  │                 │              ║          
           │                        │        ║                   │ decrypt                                      │                  │                 │              ║          
           │                        │        ║               <───┘                                              │                  │                 │              ║          
           │                        │        ║               │                                                  │                  │                 │              ║          
           │                        │        ║               │                                        guess()   │                  │                 │              ║          
           │                        │        ║               │───────────────────────────────────────────────────────────────────────────────────────>              ║          
           │                        │        ║               │                                                  │                  │                 │              ║          
           │                        │        ║               │                                                  │                  │    transfer()   │              ║          
           │                        │        ║               │                                                  │                  │ <────────────────              ║          
           │                        │        ║               │                                                  │                  │                 │              ║          
           │                        │        ║               ────┐                                              │                  │                 │              ║          
           │                        │        ║                   │ encrypt                                      │                  │                 │              ║          
           │                        │        ║               <───┘                                              │                  │                 │              ║          
           │                        │        ╚═══════════════╪══════════════════════════════════════════════════╪══════════════════╪═════════════════╪══════════════╝          
           │                        │                        │                                                  │                  │                 │                         
           │                        │                        │Controller.setEncryptedStates(GuessingGame, ERC20)│                  │                 │                         
           │                        │                        │──────────────────────────────────────────────────>                  │                 │                         
           │                        │                        │                                                  │                  │                 │                         
           │                        │   encrypted receipt    │                                                  │                  │                 │                         
           │                        │<───────────────────────│                                                  │                  │                 │                         
           │                        │                        │                                                  │                  │                 │                         
           │        receipt         │                        │                                                  │                  │                 │                         
           │<───────────────────────│                        │                                                  │                  │                 │                         
     ┌─────┴─────┐          ┌───────┴───────┐          ┌─────┴─────┐                                        ┌───┴────┐          ┌──┴──┐          ┌───┴────┐                    
     │Application│          │WalletExtension│          │ObscuroNode│                                        │Ethereum│          │ERC20│          │Guessing│                    
     └───────────┘          └───────────────┘          └───────────┘                                        │Node    │          └─────┘          │Game    │                    
                                                                                                            └────────┘                           └────────┘                    
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
W -> O: encrypted\nERC20.approve(GG)
O -> E: Controller.getEncryptedStates(ERC20)
group TEE
O -> O: decrypt
O -> T: approve()
O -> O: encrypt
end
O -> E: Controller.setEncryptedStates(ERC20)
O -> W: encrypted receipt
W -> A: receipt


==Guessing==
A -> W: GuessingGame.guess()
W -> O: encrypted\nGuessingGame.guess()
O -> E: ERC20.getEncryptedState()
O -> E: Controller.getEncryptedStates(GuessingGame, ERC20)
group TEE
O -> O: decrypt
O -> G: guess()
G -> T: transfer()
O -> O: encrypt
end
O -> E: Controller.setEncryptedStates(GuessingGame, ERC20)
O -> W: encrypted receipt
W -> A: receipt

@enduml
```
