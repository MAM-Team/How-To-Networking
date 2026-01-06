# 01 – Introduction to Networking & Unity Transport

Online games rely on one essential foundation: **network communication**. Before building gameplay systems, prediction, replication, or matchmaking, it’s important to understand what networking actually means and how Unity’s **Transport Layer** fits into the picture.

This first lesson introduces the core ideas behind networking, the difference between common communication models, and the role of Unity Transport as the low-level foundation for online games.

---

## What Is Networking?

Networking is the process of allowing two or more machines to exchange data over a communication channel.  
In games, networking is responsible for sending and receiving information such as:

- Player inputs  
- Positions and rotations  
- Health, score, and gameplay state  
- Events like shooting, jumping, or joining a lobby  

Real-time games often use lightweight, fast communication rather than heavy, reliable protocols, because every millisecond matters.

---

## Types of Network Communication

### 1. TCP (Transmission Control Protocol)
A connection-oriented, reliable protocol.

**Characteristics:**
- Guarantees delivery  
- Guarantees ordering  
- Automatically handles retransmission  
- Slower, with more overhead  
- Not ideal for real-time action games  

TCP is great for websites, APIs, or applications where accuracy is more important than speed.

---

### 2. UDP (User Datagram Protocol)
A connectionless, lightweight, fast protocol.

**Characteristics:**
- No delivery guarantee  
- No ordering guarantee  
- Minimal overhead  
- Ideal for real-time multiplayer games  

Most real-time engines rely on UDP because the game loop needs **speed** more than perfect reliability. Developers implement reliability only where needed.

---

## Client–Server Architecture

Online games almost always rely on the **Client–Server model**.

### Client
- Runs on players’ devices  
- Sends inputs (movement, shooting, actions)  
- Renders the world locally  

### Server
- Acts as the source of truth  
- Validates inputs  
- Simulates the game  
- Sends authoritative state back to all clients  

This design prevents cheating and keeps all players synchronized.

---

## What Is Unity Transport?

Unity Transport (UTP) is Unity’s low-level networking layer.  
It provides direct access to UDP-based communication while keeping performance high and overhead minimal.

Transport is the foundation underneath systems like NGO and can be used directly when you need:

- Custom protocols  
- High performance  
- Full control over packets  
- Authoritative dedicated servers  
- Lightweight, fast real-time communication  

Unity Transport gives you:

- UDP communication  
- Send/Receive pipelines  
- Connection management  
- Optional reliability  
- Network simulator (packet loss, latency, jitter)  

Transport does **not** handle gameplay logic, movement sync, prediction, or replication.  
Those systems must be built manually on top of it.

---

## Why Use Unity Transport Instead of NGO?

NGO is great for simple or small projects.  
Transport is better when you need:

- Authoritative servers  
- Custom message formats  
- Competitive shooter logic  
- Lag compensation  
- High concurrency  
- Full control over data flow  

---

## What You Will Build in This Series

This series will cover:

- A basic client–server setup  
- Message sending and receiving  
- Designing a lightweight protocol  
- Input → server → state synchronization  
- Movement and shooting demos  
- Measuring ping and simple reliability  
- Connection and lobby flow  

By the end, you will understand both the theory and the implementation of low-level multiplayer.

**02 – Creating a Client/Server Setup With Unity Transport**  
We will initialize the NetworkDriver and open a UDP connection between the client and server.
