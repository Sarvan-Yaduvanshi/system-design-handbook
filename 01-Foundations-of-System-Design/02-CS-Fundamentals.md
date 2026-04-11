# System Design — From Zero to Architect

_— Sarvan Yaduvanshi_

> Competitive Programmer · LLM Foundation Model Engineer · System Design Enthusiast

---

> [!quote] _"You cannot build on a foundation you do not understand. The internet is not magic — it is engineering. Learn the engineering."_

---

# Chapter 2 — CS Fundamentals for System Design

### _The Pillars Every System Designer Must Know_

---

## 🧱 Why This Chapter Exists

Most System Design books start directly with Load Balancers, Databases, and Caching. They assume you already know what happens when you type `https://google.com` and press Enter. They assume you know what a Thread is, what a Socket is, what TCP guarantees you.

**This book does not make that assumption.**

Before you design systems that serve millions of users, you need to understand the ground those systems stand on — how computers talk to each other, how the internet works at the wire level, and how operating systems manage the resources your code runs on.

Every single concept in this chapter will appear again — in LLD, in HLD, in case studies, and in interviews. This chapter is not an optional extra. **It is the foundation.**

> [!warning] Skip This Chapter at Your Own Risk Developers who skip networking and OS fundamentals hit a wall the moment they encounter terms like TCP handshake, race condition, I/O blocking, or TLS in a System Design interview. Do not be that developer.

---

## 🌐 Part A — Computer Networks

Everything in System Design involves data moving between machines. To design that movement intelligently, you must understand how it works.

### A1 — How the Internet Works

- What happens when you type `https://google.com` and press Enter — the full journey, step by step
- What is an IP Address — IPv4 vs IPv6
- What is a Port — why services run on specific ports
- What is a Domain Name — how `google.com` becomes an IP address
- Packets — how data is broken into pieces and reassembled
- Bandwidth vs Latency vs Throughput — the three numbers every system designer tracks

---

### A2 — The OSI Model

The seven layers of the internet. Not just theory — you will see these layers referenced in real System Design decisions constantly.

- **Layer 7 — Application** → HTTP, HTTPS, DNS, SMTP, FTP
- **Layer 6 — Presentation** → Encryption, encoding, compression
- **Layer 5 — Session** → Managing connection sessions
- **Layer 4 — Transport** → TCP, UDP — reliability vs speed
- **Layer 3 — Network** → IP addressing, routing
- **Layer 2 — Data Link** → MAC addresses, switches
- **Layer 1 — Physical** → Cables, signals, hardware

_Which layers matter most for System Design — and which you can treat as a black box._

---

### A3 — TCP vs UDP

This one decision — TCP or UDP — determines how billions of pieces of data travel across the internet every second.

- **TCP** — Transmission Control Protocol
    
    - Reliable, ordered, connection-based
    - Three-way handshake — SYN, SYN-ACK, ACK
    - Error checking, retransmission
    - Used by — HTTP, HTTPS, email, file transfers
    - When to choose TCP in your system
- **UDP** — User Datagram Protocol
    
    - Unreliable, connectionless, fast
    - No handshake, no guarantee of delivery
    - Used by — video streaming, live gaming, DNS, VoIP
    - When to choose UDP in your system
- **TCP vs UDP** — the design tradeoff table every architect knows by heart
    

---

### A4 — DNS — Domain Name System

The phonebook of the internet — and one of the most important distributed systems ever built.

- What DNS is and why it exists
- DNS Resolution — how `api.yourapp.com` becomes `192.168.1.1`
- DNS Record Types — A, AAAA, CNAME, MX, TXT, NS
- DNS Caching — TTL and why stale DNS causes incidents
- DNS Load Balancing — how DNS itself can distribute traffic
- How System Designers use DNS — geographic routing, failover

---

### A5 — HTTP and HTTPS

The language of the web. Every API you will ever design speaks HTTP.

- What is HTTP — HyperText Transfer Protocol
- HTTP Request anatomy — Method, URL, Headers, Body
- HTTP Response anatomy — Status codes, Headers, Body
- **HTTP Methods** — GET, POST, PUT, PATCH, DELETE, OPTIONS, HEAD
- **HTTP Status Codes** — 1xx, 2xx, 3xx, 4xx, 5xx — what each family means
- HTTP/1.1 vs HTTP/2 vs HTTP/3 — the evolution and why it matters for performance
- **HTTPS** — HTTP over TLS — what encryption means for your system
- Cookies, Sessions, Headers — how state is carried over a stateless protocol

---

### A6 — TLS and SSL — How Encryption Works

- What is SSL and TLS — and why SSL is deprecated
- TLS Handshake — step by step, in plain English
- Symmetric vs Asymmetric Encryption — the core idea
- Public Key, Private Key, Certificates, Certificate Authorities
- HTTPS in System Design — where TLS terminates, what a TLS terminator is
- Why encryption matters at scale — performance tradeoffs

---

### A7 — WebSockets and Real-Time Communication

HTTP is request-response. But some systems need to push data continuously — chat apps, live scores, stock prices, notifications. That requires a different protocol.

- The problem with HTTP for real-time — why polling is inefficient
- **Short Polling** — ask repeatedly, wasteful but simple
- **Long Polling** — hold the connection open, a step better
- **Server-Sent Events (SSE)** — one-way push from server to client
- **WebSockets** — full-duplex, persistent, bidirectional connection
    - The WebSocket handshake
    - When to use WebSockets vs SSE vs polling
    - Real examples — WhatsApp Web, Google Docs live cursor, Zerodha live prices

---

### A8 — APIs — The Contracts Between Systems

- What is an API and why it is the fundamental building block of modern systems
- **REST** — Representational State Transfer
    - Principles — stateless, uniform interface, resource-based
    - REST vs CRUD — they are not the same thing
    - RESTful URL design — best practices and common mistakes
- **GraphQL** — query exactly what you need, nothing more
    - When GraphQL wins over REST
    - N+1 problem and how GraphQL solves it
- **gRPC** — Google's high-performance RPC framework
    - Protocol Buffers — binary serialization
    - Why gRPC is used for internal microservice communication
    - gRPC vs REST — latency and throughput comparison
- **Webhooks** — reverse APIs, event-driven callbacks
- **API Versioning** — how to evolve APIs without breaking clients

---

### A9 — Proxies and Reverse Proxies

- **Forward Proxy** — client-side, hides the client from the internet
    - VPNs, corporate proxies, content filtering
- **Reverse Proxy** — server-side, hides servers from clients
    - Nginx, HAProxy as reverse proxies
    - SSL termination, caching, compression at the proxy level
- Why reverse proxies are everywhere in production systems
- **API Gateway** — the intelligent reverse proxy for microservices

---

### A10 — Network Fundamentals That Show Up in Interviews

- **Latency numbers every engineer should know** — RAM, SSD, network, cross-datacenter
- **CAP Theorem preview** — consistency, availability, partition tolerance
- **Idempotency** — why it matters for APIs and distributed systems
- **Timeouts and Retries** — how systems handle network failures gracefully
- **Circuit Breaker pattern** — stopping failure from cascading
- **Long tail latency** — the p50, p95, p99 problem

---

## 💻 Part B — Operating System Concepts

Your code does not run in a vacuum. It runs on an operating system that manages CPU, memory, disk, and network on your behalf. Understanding the OS is what allows you to design systems that are fast, concurrent, and reliable.

---

### B1 — Processes and Threads

- **Process** — an isolated running program with its own memory space
- **Thread** — a lightweight unit of execution within a process, sharing memory
- Process vs Thread — isolation, overhead, communication
- **Multiprocessing vs Multithreading** — when to use which
- **Context Switching** — the cost of the OS juggling many threads
- How web servers use processes and threads — Apache vs Nginx model

---

### B2 — Concurrency and Parallelism

Two concepts that sound the same but are completely different — and both matter enormously in System Design.

- **Concurrency** — dealing with many things at the same time (structure)
- **Parallelism** — doing many things at the same time (execution)
- Single-core concurrency — how one CPU handles thousands of tasks
- Multi-core parallelism — true simultaneous execution
- **Async / Non-blocking I/O** — how Node.js handles 10,000 connections on one thread
- **Event Loop** — the mechanism behind async programming
- Thread pools — why servers do not create a new thread per request

---

### B3 — Synchronization — Locks, Mutex, and Semaphore

When multiple threads share data, things go wrong. This section explains what goes wrong and how to fix it.

- **Race Condition** — two threads writing at the same time, corrupting data
- **Critical Section** — the code that must not run concurrently
- **Mutex (Mutual Exclusion Lock)** — only one thread enters at a time
- **Semaphore** — control access for N threads simultaneously
- **Deadlock** — two threads waiting for each other forever
    - Four conditions that cause deadlock
    - How to prevent and detect deadlock
- **Livelock** — threads keep responding to each other but make no progress
- **Starvation** — a thread is always waiting, never runs
- How these concepts appear in database locks, distributed locks, and queue design

---

### B4 — Memory — How Computers Store Data

- **Memory Hierarchy** — Registers → L1/L2/L3 Cache → RAM → SSD → HDD → Network
- Why the hierarchy matters — latency at each level
- **Stack vs Heap** — where variables live and why it matters
- **Virtual Memory** — how the OS gives each process the illusion of infinite RAM
- **Paging and Swapping** — what happens when RAM fills up
- **Memory Leaks** — what they are and how they kill production services
- Cache-friendly code — why data layout in memory affects performance

---

### B5 — Storage — Disk, Files, and I/O

- **HDD vs SSD** — mechanical vs flash, latency and throughput numbers
- **Sequential vs Random I/O** — why databases care deeply about this
- **File System Basics** — how files, directories, and inodes work
- **Buffered vs Unbuffered I/O** — flushing, fsync, durability
- **Memory-Mapped Files** — how databases like SQLite and Kafka use them
- **RAID** — redundancy and performance in disk arrays
- **Block Storage vs File Storage vs Object Storage** — the three storage models used in cloud systems

---

### B6 — Caching at the OS Level

Before you understand Redis and CDNs, understand where caching actually begins.

- **CPU Cache** — L1, L2, L3 — the fastest memory your program can touch
- **Page Cache** — the OS caching disk reads in RAM automatically
- **Buffer Cache** — write buffering before hitting disk
- **Why caching is the single biggest performance lever in any system**
- Cache hit vs cache miss — the numbers that determine system speed
- This is the mental model you carry into every application-level caching decision

---

### B7 — Concurrency Patterns Used in System Design

- **Producer-Consumer Pattern** — one side creates work, another processes it — the foundation of message queues
- **Reader-Writer Lock** — many readers, one writer — used in databases
- **Thread Pool Pattern** — fixed set of workers processing a queue of tasks
- **Async / Await and Futures** — language-level concurrency primitives
- **Actor Model** — each actor has its own state, communicates only by messages — used in Erlang, Akka, and distributed systems

---

### B8 — OS Concepts That Show Up in System Design Interviews

- **File Descriptors and Sockets** — everything in Linux is a file
- **Epoll / Kqueue** — how modern servers handle thousands of connections efficiently
- **Signals and Interrupts** — how the OS notifies processes of events
- **Fork and Exec** — how new processes are born
- **System Calls** — the boundary between your code and the OS kernel
- **The C10K Problem** — how to handle 10,000 concurrent connections and the OS-level solutions that made it possible

---

## 📋 Chapter 2 — Full Table of Contents

```
🌐  PART A — COMPUTER NETWORKS
    ├── A1  How the Internet Works
    │       IP Addresses · Ports · Packets · Bandwidth vs Latency
    ├── A2  The OSI Model — All 7 Layers
    │       What each layer does · Which layers matter for System Design
    ├── A3  TCP vs UDP
    │       Three-way Handshake · Reliability · When to choose which
    ├── A4  DNS — Domain Name System
    │       Resolution · Record Types · Caching · TTL · DNS Load Balancing
    ├── A5  HTTP and HTTPS
    │       Methods · Status Codes · HTTP/1.1 vs HTTP/2 vs HTTP/3
    ├── A6  TLS and SSL — Encryption
    │       TLS Handshake · Certificates · Symmetric vs Asymmetric
    ├── A7  Real-Time Communication
    │       Short Polling · Long Polling · SSE · WebSockets
    ├── A8  APIs — REST, GraphQL, gRPC, Webhooks
    │       Principles · When to use which · API Versioning
    ├── A9  Proxies and Reverse Proxies
    │       Forward Proxy · Reverse Proxy · API Gateway
    └── A10 Network Concepts in Interviews
            Latency Numbers · Idempotency · Timeouts · Circuit Breaker

💻  PART B — OPERATING SYSTEMS
    ├── B1  Processes and Threads
    │       Isolation · Context Switching · Web server models
    ├── B2  Concurrency and Parallelism
    │       Async I/O · Event Loop · Thread Pools · Node.js model
    ├── B3  Synchronization
    │       Race Condition · Mutex · Semaphore · Deadlock · Starvation
    ├── B4  Memory
    │       Stack vs Heap · Virtual Memory · Paging · Memory Leaks
    ├── B5  Storage and Disk I/O
    │       HDD vs SSD · Sequential vs Random I/O · Block vs Object Storage
    ├── B6  Caching at the OS Level
    │       CPU Cache · Page Cache · Cache Hit vs Miss
    ├── B7  Concurrency Patterns
    │       Producer-Consumer · Reader-Writer · Actor Model
    └── B8  OS Concepts in Interviews
            File Descriptors · Epoll · The C10K Problem
```

> [!success] After This Chapter Every term you encounter in LLD and HLD will have a foundation. When someone says "the service is I/O bound" or "we hit a race condition in production" or "we need persistent WebSocket connections" — you will not just nod. You will understand exactly what is happening and why.

---

_Next →_ [[02-Foundations-of-System-Design/Network-How-Internet-Works|Chapter 2.1 — How the Internet Works]]

---

_— Sarvan Yaduvanshi · Competitive Programmer · LLM Foundation Model Engineer_