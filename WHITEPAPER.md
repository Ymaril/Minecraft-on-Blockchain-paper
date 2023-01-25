# Minecraft on Blockchain
**A Blockchain to ensure the operation of a fully functional minecraft world**

_NOTE: This document is under development. Please check regularly for updates!_

## Table of Contents
- [Motivation](#motivation)
  * [Minecraft Anarchy](#minecraft-anarchy)
  * [2Builders 2Tools](#2builders-2tools)
  * [Problems of Minecraft Anarchy](#problems-of-minecraft-anarchy)
- [Solution requirements](#solution-requirements)
- [Terminology](#terminology)
- [Similarity to cryptocurrencies](#similarity-to-cryptocurrencies)
  * [Complexity of transaction validation](#complexity-of-transaction-validation)
  * [Consensus](#consensus)
  * [Summary](#summary)
- [Solution](#solution)
  * [Layers](#layers)
  * [World storage and processing layer](#world-storage-and-processing-layer)
  * [Player interaction layer](#player-interaction-layer)
- [Potential commercialization, tokenization](#potential-commercialization-tokenization)

## Motivation
### Minecraft Anarchy
A popular way to play Minecraft. Minecraft Anarchy is a game without rules that allows grief, stealing and killing other players.
The game uses vanilla version of Minecraft so there is no privacy (protected territoriy) and no teleportation abilities
### 2Builders 2Tools
2Builders 2Tools (2B2T) is the most prominent example of Minecraft Anarchy genre
This server exists since 2010 and has done no world resets

Eternal world gave players opportunities for ambitious projects and long-term plans
Server epochs are defined by various events such as clan rises, overthrow of leaders, hacks, griefs and so on. The eternal world has its history with rare things and unique memorials

### Problems of Minecraft Anarchy
_Using 2B2T example_

Server is often unavailable. Sometimes it happens because of scheduled maintenance, sometimes because of failures

Server’s admin, Housemaster, tries to maintain the server operable and avoids abusing his powers. However, there have been cases when he interfered and removed or restored structures. Sometimes it was reasonable, like in case with lag machines that slow down the server and ruin fun for other players

The server often lags and it impedes gameplay even if players are physically located away from other players

Any Minecraft server has standard problems of client-server architecture

## Solution requirements
* Decentralized solution where anyone can create a node
* The world must be eternal (stored between all network participants)
* The world should be protected from fraud, cheating and hacking, need a capability to validate the world and players
* The world must be based on rules of vanilla Minecraft
* The world must have a closed coordinate system. Network members should not have free access to player coordinates and shouldn’t be able to view and analyze the entire world
> Hidden construction is an important aspect of Minecraft Anarchy. Players walk to distant coordinates and cover their traces hoping that their structure stays unfound. It’s necessary to maintain the ability to hide construction and structures
* Players must be able to interact in real time
* No admins. All players must have the same authority and play by the same rules


## Terminology
* **Node** – a Minecraft server capable of communicating with other similar nodes within a network. Players use vanilla Minecraft client to connect to this node. A node can serve multiple players
* **Player** – a user connecting to a node with a vanilla Minecraft client. Players can interact with the world and other players in the world
* **World** – a Minecraft world is a place where players interact with each other and the environment
* **Chunk** – a square area in vanilla Minecraft world (16x16 blocks). Chunks make storage and synchronization more convenient
* **Blockchain** – a chain of blocks. Non-alterable verifiable database
* **DAG** – directed acyclic graph
* **Block** – a part of blockchain or DAG that stores data of world and player changes. Sometimes blocks can have values from Minecraft world

## Similarity to cryptocurrencies
Most ideas and technologies required for implementation already exist and are used for decentralized finance problems

Cryptocurrency blockchains store a history of user interaction. It usually takes the form of transactions containing information of participants, amount and sender’s signature.
With local storage transactions are validated by network participants. If a sender doesn’t have enough money or if a transaction has an invalid signature, the transaction is considered invalid

Minecraft On Blockchain also uses transactions describing interaction of players with the world. For example: destroyed a block, placed a block, pick up an item, drop an item

### Complexity of transaction validation
Validation of transactions describing Minecraft world interaction is a labor intensive task because it has to apply all Minecraft rules

Some implementations of deFi platforms use smart contracts — a more complex set of custom transaction validation rules

Technically, smart contracts can be used to implement the entire Minecraft logic. However, this is not a good solution

* It is too hard to move all mechanics of a Minecraft world into a smart contract. The only option is to rewrite it, which is also a labor intensive task
* Smart contract deployment and interactions with smart contracts cost money. The more complex is a smart contract, the higher are the costs. In our case those costs are prohibitive
* Network transactions are not instant. It is impossible to achieve quick response of the world and other players. For example, block destruction transaction may take several minutes, so it will be simply impossible to play

To solve this problems we are going to use an additional “player interaction layer”

### Consensus
Nodes in proof of work blockchains validate every incoming transaction. It means that all network participants execute the same validation
It is acceptable for simple transactions but not in our case

Another option is proof of stake consensus. It doesn’t require nodes to validate blocks and transactions, they can just verify signatures of validators
This principle is more suitable for complex transactions because actual validation will be done only by validators that are rewarded for it

### Summary
Minecraft on Blockchain is a cryptocurrency with complex transactions, so it is preferrable to use Proof of Stake consensus for it

## Solution
### Layers
At first approximation, the solution will have two high-level layers:
* World storage and processing layer
> * World storage, validation of user interactions, synchronization of changes
> * Proof Of Stake blockchain platform
> * Exchange and storage of blockchain with world changes
> * Creation and validation of new blocks with transactions
>
> _Virtually everything is implemented similarly to cryptocurrencies_
* Players interaction layer
> * Direct interaction of players that are located near each other
> * Support of direct secure connection and data exchange between two nodes
> * Verifying adjacence of player locations without sending actual player coordinates to the network

### World storage and processing layer
Minecraft on Blockchain uses separates blockchains to storage every chunk of the world

Vanilla Minecraft doesn’t store the entire world in memory. Minecraft world generator purposedly creates chunks that are visited by the player. Unchanged chunks can be regenerated upon next player visit without storing them in memory. Storing makes sense only if a chunk has been changed

A similar scheme is envisaged for Minecraft on Blockchain. World seed is open to all network participants; they will be able to generate non-existent chunks

When a player interacts with a chunk, blockchain creates a block describing changes of a chunk with relative coordinates from chunk start point
> For example: destroyed block: x: 4, y: 64, z: 2

If this is a first change, it can be the first blockchain block (genesis block)

#### Blockchain block identification
Block ID is hash sum of the sum of coordinates and the hash of new state of a chunk. The formula looks like this:
```
Hash(Chunk coordinates + Hash(New state))
```
The world must use a hidden coordinate system. Network participants may not have free access to player coordinates, view and analyze the entire world. Blockchain should not store chunk coordinates in open form

A node doesn’t know new chunk state, so it needs to find a hash sum of this state. It will be a proof of work task making it impossible to freely scan the world for buildings. A node will look through blockchains of recent chunks and compare them with hash sum. This way it will sooner or later find the desired chunk blockchain

A node needs only chunks that surround the player. It needs real coordinates to receive this information. Therefore, it collects hash of surrounding chunk coordinates and brute-force a new state hash. World scanners will need to perform these tasks to bind blockchain with real coordinates, and these tasks are likely to be too heavy for them

#### Chunk encryption
You can pick the right hash for chunk’s blockchain only once. After picking you need to store it and get the ability to track changes without any problems

This process is made even harder with encryption of every block in chunk’s blockchain. So, certain work is needed to get current chunk state (proof of work).
Encryption key is based on hash of new chunk state (as in case with ID). There is no need to find a key again if it is already available on ID level

The amount of work is small enough to make chunk loading time acceptable for honest users (while player moves from one border of a chunk to another border). However, it is hard to collect a lot of chunks and there is little point in doing so as most of the chunks will be empty

Each block of chunk’s blockchain contains encryption key of the previous block. You need only one last key to access all content’s of the chunk. This ensures fixed nature of work.

#### Transaction
_Any player interaction with the world_

Contains fields:
* tick
> Something like a timestamp. A tick is a unit of time in vanilla Minecraft. Everything that happens, happens on a certain tick. It is a discrete time unit. In transaction it means event date
event
* event_type 
> Content depends on even type
* coordinates
> All coordinates are given relatively to beginning of the chunk
* player _(optionaly)_
> If event contains a player, it needs the player’s signature
chunk
* chunk_id
* signature
> Player’s signature of transaction

### Player interaction layer
Players need the ability to meet and interact in the world

Nodes need make a direct connection for such interaction. However, this is not an option as it poses a risk of excessive network load. Besides, world’s coordinate system must be hidden. Network participants should not have free access to player coordinates, view and analyzing the world

Two nodes need the ability to prove each other that their players are nearby without disclosing the coordinates and ensure consistence

Each time when a user enters a chunk, the node announce it sending chunk block ID to the network. Each other node checks whether this chunk is located in load area of any other player. If verification is successful, the requested node sends a confirmation to the requestor node and transmits chunk block ID of its player. The requestor node executes similar verification and establishes direct connection

## Potential commercialization, tokenization
The project is not primarily aimed for commercialization. However, there is a potential of game item price increase.
Moreover, network development requires a local currency to pay fees and validator rewards

Vanilla Minecraft servers often use diamonds as currency. They are not replenishable and their quantity in Minecraft world is restricted.
So, nothing prevents using diamonds as native currency in Minecraft on Blockchain.
It is possible to restrict validator rewards in diamonds

Diamonds can be preminted.
First players will have more opportunities to get a lot of diamonds
First validators will get the highest rewards
If the project becomes popular, diamond value can increase