# Motivation
Could Obscuro be turned into an application of an Ethereum-compatible network, instead of being an L2 chain or a side-chain? It might have the following advantages:
1. Easier adoption.
2. Ability to compose privacy-enabled contracts with existing regular contracts.
3. No rollup costs, instead individual transaction costs.
4. Work with any EVM network, e.g. Telos, Fantom, Avalanche C, Polygon PoS, etc.

# Editing Diagrams
Diagrams may be edited using an online PlantUML editor, such as [https://plantuml.com/](https://plantuml.com/).

# Basic Interaction
![Basic Interaction](https://www.plantuml.com/plantuml/svg/TPC_SzD04CNx-nJBq2P566XSC76C6YPZPQ1GlJcjhPj8jyVk2lDd-ExeR0dBH2xrVk-zUrleNWt5-gvrRvBWlaCmubzWqfFJbn0J2dRGSMJV27S4EsnrZeJxM7kMI09t7sP06wpv4EB-LKJfMq_Hqsy7i1RXmuPR5dXRREu-lNi_Y4ye5dn86Eq1_Sl--CR9L3N1w3yBIqroYRTiT2sQsJppq0x6FCKRorLmhUqnxEWnn8N6FxEJ8zlntANwnUOXxTBvHbYdr0QF5ZW2AgmlO8LjIvRrQa4lVXX57ODLSU4edzNtmbkkATFo0XRMa53VPL8ubsy0_au2vGqKAz0-9HRJi-_prt9x--LfuzPm-n6g5GWUm0IZei_BNJNg6lRJnr14qSlFPAugibAke0wQq7pZoYmCZXqDN-2FAsCDQcZaoj6TX3sfdjIRVsrJLlldBXX4EeovwHKXJs4opdPGFQ5nKNNskdkATVvR_W80)
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
![Guessing Game Example](https://www.plantuml.com/plantuml/svg/bPJDRjim48JlV8fjUac19cY2zcB0HHoN4504KP0OqCKNLbhRX2Kkkrmb_K7UlQKbnMAX77eqlvaTpjBGLH33qZQrUkFPR1i3WlGBX5jKOM-TDsmQQCXHkMDg43Qc3cF4rBgg97R8SF5f1CBELFHatJQrH8BVMtGb_bO02sPwMb9D36_rykBBvKS5S6TOR6czSN9joltsaD69YfqL_4T2N26d-9O4KslAnfxh-gymNgKKujV1AzDkx0Plh9kO-3ogx4WmrfqQFptvMIb_x0WBxsjR6h7am0HO9B0mTOsYVmYwq0kAH91_2sNBT4iLHfWdy4JB3XdRPksYu2GgTABGzylB3_5ubHgp5aEmRhjscMawOwPKx0hqY0pvUz0p2Attn19Wo86l3_djsVtTKgdvV4LxTB4UqwEfvtErWBFFKCowq7Dp07YQPUzK4Ka-0tGb__I2rTgDSHwndZ1sDwfn8pq46-AB70Sf7Rsh6OnTkezZBD5wwUONe_Y1T1TR79Mnrx2RKQFXA2xhV1j_d4wKWe4CVL8RYXwK3s84JvDwOsriUMmO_s4ULbghl4-iCBA-LpPxoStwNGWR5pR8QS9m2jO9Sec7uTdBge8d5r3bQIk5hfhk4oRwf0yvkiP_lC0LkYf-rVu0)
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
![Moving World State Into TEE](https://www.plantuml.com/plantuml/svg/fP1FJzj04CNl-occz08Ie8BQYwLKXDIoL26C898lkMpjYRFAUcSxEuR-4T-zh8DRkpWILO-xx_VcpRoB89gbRQnwurdNZGP1-W72BQgmDSvhrWqqvAZSC3K8Qst3K69gNLKIEsGu-p0GmaPNz61S_S8Iy6kBhiIV2c3EJ0yBiiJmDbsSVJf_hm1kD8ifZTTEBdMvhRkC7LHiZE1V2765F9QlIN1i7Mj3pwLz0kCgfH3l5VPDkp9hl39kOkBZu6H7qfqxDNvwygDI_i-DCFVUcbAB8GSwm7ma5jfQbFIRe0lFG342yY-XR9ckgS8evYFn89KDChRDoaNW81BfY4Xl5-VlulToRCnQ3C6uUkKotUoCcLBnLg1xPCYF8KqWjDuJIu0-1pxVvTSdjpSBfMQp9MtHnVEOxhMTpTGSJZv2aUn2JtKdu66M7QaYAda2w4h-xWMhbNi9vKzGJ5sG4oPhaKzhbBItt8aM38UTxqZ5sZqiqxIpnTaLTk3klKoW7nq7Zz8LkcegBEmDw-1Yi3umbcYyx0wR9x_ViPU-sq4Z_LSi-xIEZyF9KYQvr_Koj1x3wrhA4kX3ikVMXBKBQ-J_gcsyox-NU86kQXlx2m00)
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
How is this different from the current Obscuro design?
1. The Obscuro network of nodes is stateless, i.e. the necessary world view accounts (data and code) are retrieved at the point of execution.
2. There is no rollup of transactions representing the delta changes to the world view; instead, individual updated accounts are persisted after each transaction.
