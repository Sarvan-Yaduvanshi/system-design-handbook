# System Design — From Zero to Architect

_— Sarvan Yaduvanshi_

> Competitive Programmer · LLM Foundation Model Engineer · System Design Enthusiast

---

> [!quote] _"Your code is a guest. The operating system is the host. Know your host."_

---

# Chapter 2 — CS Fundamentals for System Design

## Part B — Operating System Concepts

---

## 💻 B1 — Processes and Threads

Every program you run — your browser, your server, your database — runs as a **process**. Inside that process, the actual work is done by **threads**. Understanding this distinction is the foundation of everything in concurrent system design.

---

### What is a Process?

A **process** is an independent, isolated running instance of a program. When you start your Node.js server, the OS creates a process for it. That process gets its own:

- **Memory space** — its own RAM allocation, isolated from all other processes
- **File handles** — its own open files and network connections
- **Process ID (PID)** — a unique number the OS uses to manage it

Because processes are isolated, **one process cannot accidentally corrupt another's memory**. If your server process crashes, your database process is unaffected. This isolation is both the strength and the cost of processes.

> [!example] Real Example Open your Mac or Linux terminal and type `ps aux`. Every line is a running process — your browser, your editor, your terminal, background daemons. Each completely isolated from the others.

---

### What is a Thread?

A **thread** is a unit of execution that lives _inside_ a process. One process can have many threads. All threads in the same process **share the same memory**.

Think of a process as a restaurant kitchen. The kitchen itself (memory, equipment, ingredients) is the process. Each chef working in that kitchen is a thread — they all share the same space and resources, and they can all work simultaneously.

**Because threads share memory:**

- Communication between threads is fast — just read/write shared memory
- But — if two threads write to the same memory at the same time, data gets corrupted. This is the problem that Sections B3 covers in full.

---

### Process vs Thread — The Key Differences

||**Process**|**Thread**|
|---|---|---|
|**Memory**|Own isolated memory|Shared with other threads|
|**Creation cost**|Heavy — OS allocates new memory space|Light — created within existing process|
|**Communication**|Slow — IPC (pipes, sockets, shared memory)|Fast — directly read/write shared memory|
|**Crash impact**|Crash does not affect other processes|Crash can bring down the entire process|
|**Use for**|Isolation, independent services|Parallel tasks within one service|

---

### Context Switching — The Hidden Cost

Your laptop has 8 cores but runs hundreds of processes simultaneously. How? The OS rapidly switches between them — giving each a tiny slice of CPU time, so fast it feels simultaneous. This is called **context switching**.

When the OS switches from Thread A to Thread B, it must:

1. Save Thread A's entire state — registers, program counter, stack pointer
2. Load Thread B's saved state
3. Resume Thread B

This takes time — typically **1–10 microseconds** per switch. That sounds tiny. But if your server creates a new thread for every incoming request, and you have 10,000 concurrent requests, you are spending more CPU time switching contexts than doing actual work.

> [!warning] The Practical Implication This is exactly why modern high-performance servers do not create one thread per connection. Thread pools and async I/O (covered in B2) exist specifically to avoid this.

---

### How Web Servers Use Processes and Threads

**Apache — Process-per-request (old model)** Apache historically spawned a new process for each connection. Extremely isolated. But processes are heavy — each consumed megabytes of RAM. At 10,000 simultaneous connections, Apache used gigabytes just for process overhead. This is the **C10K problem** (covered in B8).

**Nginx — Event-driven, single-threaded (modern model)** Nginx uses a small, fixed number of worker processes — typically one per CPU core. Each worker handles thousands of connections using async I/O and an event loop. No new thread or process per connection. Nginx serves millions of requests per second on modest hardware. This architecture is why Nginx replaced Apache as the dominant web server.

---

## 🔀 B2 — Concurrency and Parallelism

These two words are used interchangeably in casual conversation. In engineering, they mean completely different things. Confusing them leads to wrong architectural decisions.

---

### Concurrency vs Parallelism — The Precise Definitions

**Concurrency** is about _structure_ — designing a program to handle multiple tasks by interleaving their execution. The tasks may not actually run at the exact same instant.

**Parallelism** is about _execution_ — multiple tasks genuinely running at the same physical instant on multiple CPU cores.

> [!example] The Coffee Shop Analogy **Concurrency:** One barista taking Order A, starting to brew it, then taking Order B while A brews, then finishing A when ready. One person, multiple tasks in progress simultaneously. **Parallelism:** Three baristas each independently brewing a different order at the exact same moment. Multiple people, multiple tasks executing simultaneously.
> 
> You can have concurrency without parallelism (one CPU, many tasks). You can have parallelism without good concurrency design (multiple CPUs, poorly coordinated). The best systems have both.

---

### Single-Core Concurrency

A single CPU core executes exactly one instruction at a time. So how does your phone run 50 apps "simultaneously"?

The OS **time-slices** — it runs App A for 10ms, pauses it, runs App B for 10ms, pauses it, runs App C... cycling through so fast that everything appears simultaneous. This is concurrency without parallelism.

The critical insight: **most tasks spend most of their time waiting** — waiting for a network response, waiting for a disk read, waiting for user input. While one task waits, another runs. Concurrency exploits this waiting time.

---

### Async / Non-Blocking I/O

Traditional (blocking) I/O:

```
Thread asks disk for data → Thread WAITS → Disk delivers → Thread continues
```

The thread is blocked — doing nothing — while waiting for the disk or network. If you have 1,000 threads waiting on I/O, you have 1,000 threads consuming memory and context-switch overhead while contributing zero work.

Non-blocking I/O:

```
Thread asks disk for data → Thread goes and does other work → OS notifies when data is ready
```

The thread never waits. It kicks off an I/O operation, registers a callback, and moves on to handle the next request. When the data is ready, the OS triggers the callback.

This is how **Node.js serves 10,000 connections on a single thread**. It never blocks. Every I/O operation is async. The single thread is always doing useful work.

---

### The Event Loop

The Event Loop is the engine behind async programming. It is how a single thread manages thousands of concurrent operations.

```
┌─────────────────────────────────┐
│           Event Loop            │
│                                 │
│  1. Take next task from queue   │
│  2. Execute it until completion │
│     or until it hits async I/O  │
│  3. Register callback for I/O   │
│  4. Move to next task           │
│  5. When I/O completes, OS      │
│     puts callback in queue      │
│  6. Go to step 1                │
└─────────────────────────────────┘
```

The event loop never blocks. It processes tasks from a queue, and when a task triggers I/O, it does not wait — it registers what to do when that I/O completes and moves on immediately.

> [!tip] Where You See This JavaScript (Node.js), Python (asyncio), Go (goroutines), and Nginx all use event-loop or event-driven architectures for exactly this reason — maximum throughput with minimal threads.

---

### Thread Pools — The Practical Solution

Creating a new thread for each task is expensive (memory allocation, OS registration, context switch overhead). The solution is a **thread pool** — a fixed set of pre-created threads that sit waiting for work.

```
Incoming requests → Task Queue → [Thread 1] [Thread 2] [Thread 3] [Thread 4]
```

When a request arrives, it is placed in the queue. The next available thread picks it up. When finished, the thread returns to the pool — ready for the next task. No creation, no destruction. Fixed, predictable resource usage.

Most production servers, databases, and executors use thread pools internally. Java's `ExecutorService`, Python's `ThreadPoolExecutor`, and database connection pools are all implementations of this pattern.

---

## 🔒 B3 — Synchronization — Locks, Mutex, and Semaphore

Threads sharing memory is powerful. It is also dangerous. This section is about what goes wrong when threads share data carelessly — and the tools that make it safe.

---

### Race Condition

A **race condition** occurs when two or more threads access shared data simultaneously, and the final result depends on the timing of their execution.

**Classic example — two threads incrementing a counter:**

```
counter = 100

Thread A reads counter → gets 100
Thread B reads counter → gets 100
Thread A adds 1 → writes 101
Thread B adds 1 → writes 101    ← Should be 102!
```

Both threads read the value before either wrote back. Both added 1 to 100. The counter ends at 101 instead of 102. **One increment was lost.**

Scale this to a bank — two simultaneous withdrawals, both reading the same balance before either deducts. Both succeed. Your account goes negative. This is a race condition with real consequences.

> [!warning] The Nasty Truth About Race Conditions Race conditions do not happen consistently. They happen based on timing — which varies with CPU load, OS scheduling, and hardware. They pass your tests. They appear in production, at scale, under load, at the worst possible moment. They are one of the hardest bugs to reproduce and debug.

---

### Critical Section

A **critical section** is a block of code that accesses shared data and must not be executed by more than one thread at a time.

The solution to a race condition is to protect the critical section — ensure only one thread can be inside it at any given moment. This is called **mutual exclusion**, and the tool that enforces it is a **mutex**.

---

### Mutex — Mutual Exclusion Lock

A **mutex** is a lock. Before entering the critical section, a thread must acquire the lock. After exiting, it releases the lock. If another thread tries to acquire a held lock, it blocks — waits — until the lock is released.

```
Thread A: acquire lock → read counter (100) → add 1 → write (101) → release lock
Thread B:              → tries to acquire lock → BLOCKED → waits...
                                                            → lock released
                                                            → acquires lock
                                                            → read counter (101) → add 1 → write (102) → release
```

Counter correctly reaches 102. Race condition eliminated.

**The cost:** Threads waiting for a lock are not doing useful work. In high-contention scenarios — many threads competing for the same lock — performance degrades. **Lock granularity** is a real engineering decision — coarse locks are simple but contended, fine-grained locks are fast but complex.

---

### Semaphore

A **semaphore** is a generalised mutex. Where a mutex allows exactly **1** thread into the critical section, a semaphore allows exactly **N** threads simultaneously.

Think of a semaphore as a counter representing available permits. A thread acquires one permit to enter, releases it when done.

**Binary semaphore (N=1):** identical to a mutex.

**Counting semaphore:** controls a pool of resources.

> [!example] Real Example — Database Connection Pool Your application has a pool of 10 database connections. A semaphore with N=10 guards access. The 1st through 10th threads acquire connections and proceed. The 11th thread tries to acquire — no permits left — it blocks and waits. When any of the first 10 finish and release their connection, the 11th thread wakes up and proceeds. The pool never exceeds 10 connections. The database is protected from being overwhelmed.

---

### Deadlock

A **deadlock** occurs when two or more threads are each waiting for a resource held by the other — and none can proceed. Everything freezes. Permanently.

**Classic two-thread deadlock:**

```
Thread A holds Lock 1, waiting for Lock 2
Thread B holds Lock 2, waiting for Lock 1

Both wait forever. Neither proceeds. System hangs.
```

**The four conditions for deadlock (Coffman Conditions):** All four must hold simultaneously for a deadlock to occur:

1. **Mutual Exclusion** — at least one resource is non-shareable (only one thread at a time)
2. **Hold and Wait** — a thread holds at least one resource while waiting for another
3. **No Preemption** — resources cannot be forcibly taken from a thread
4. **Circular Wait** — Thread A waits for Thread B, which waits for Thread A

**How to prevent deadlock:** Break any one of the four conditions.

The most practical approach: **always acquire locks in the same global order**. If every thread always acquires Lock 1 before Lock 2, circular wait is impossible. Thread B can never hold Lock 2 while waiting for Lock 1, because it would have had to acquire Lock 1 first.

---

### Livelock

A **livelock** is like a deadlock, except the threads are not stuck — they are active. But they keep responding to each other without making any actual progress.

> [!example] The Hallway Analogy Two people walking toward each other in a corridor. Both step right to let the other pass. Both now step left. Both step right again. They are both moving — actively trying to be polite — but neither gets past. This can continue indefinitely.
> 
> In software: Thread A detects conflict with Thread B and backs off. Thread B detects conflict with Thread A and backs off. Both retry at the same moment. Conflict again. Both back off again. Forever.

**Solution:** Randomised backoff — each thread waits a random amount of time before retrying, ensuring they do not stay in sync.

---

### Starvation

**Starvation** occurs when a thread is perpetually denied access to a resource it needs — not because of deadlock, but because other threads are always prioritised over it.

> [!example] Low-Priority Thread A background analytics thread has low priority. Every time the CPU is free, high-priority web request threads take it first. The analytics thread is always ready to run but never gets scheduled. It is technically alive but functionally starved.

**Solution:** Fair scheduling algorithms that guarantee every thread eventually gets access, regardless of priority.

---

### How These Concepts Appear in System Design

- **Database row locks** — when a transaction updates a row, it acquires a mutex on that row. Other transactions wanting the same row wait. This is exactly the mutex concept at the database level.
- **Distributed locks** — in a microservices system, multiple instances might try to process the same job. A Redis-based distributed lock (using `SET NX`) acts as a mutex across machines.
- **Message queue consumers** — a semaphore-like mechanism ensures only N workers process messages concurrently, protecting downstream services from being overwhelmed.

---

## 🧠 B4 — Memory — How Computers Store Data

Memory is not a single thing. It is a hierarchy — multiple layers of storage, each faster but smaller and more expensive than the next. Understanding this hierarchy is what allows you to write fast code and design fast systems.

---

### The Memory Hierarchy

```
Fastest │  Registers        ~0.3 ns    bytes
        │  L1 Cache         ~1 ns      32–64 KB per core
        │  L2 Cache         ~4 ns      256 KB–1 MB per core
        │  L3 Cache         ~10 ns     4–64 MB shared
        │  RAM              ~100 ns    8–128 GB
        │  SSD              ~150 µs    256 GB–4 TB
        │  HDD              ~10 ms     1–20 TB
Slowest │  Network          ~100 ms    unlimited
```

Each level is roughly **10–1000x slower** than the one above it.

The practical implication: **where your data lives determines how fast your program runs**. Data in L1 cache is retrieved in 1 nanosecond. The same data on a remote server takes 100 milliseconds — 100 million times slower.

---

### Stack vs Heap

Inside a process's RAM, memory is organised into two main regions:

**The Stack**

- Stores local variables and function call information
- **Fixed size**, allocated at program start
- **Automatically managed** — when a function returns, its stack frame is popped off instantly
- **Very fast** — just moving a stack pointer
- **Limited size** — typically 1–8 MB. Exceed it → stack overflow crash

**The Heap**

- Stores dynamically allocated data — objects created at runtime
- **Flexible size** — grows and shrinks as needed
- **Manually managed** (in C/C++) or **garbage collected** (in Java, Python, Go)
- **Slower** than the stack — allocation requires finding free space
- **Larger** — limited only by available RAM

> [!example] Simple Rule `int x = 5` inside a function → stack. `new User()` or `malloc()` → heap. The stack is for short-lived, fixed-size data. The heap is for everything else.

---

### Virtual Memory

Every process believes it has access to a large, contiguous block of memory — often the entire 64-bit address space (16 exabytes in theory). This is a lie the OS tells, and it is a very useful lie.

**Virtual memory** is an abstraction layer. The OS maps each process's virtual addresses to physical RAM addresses. A process at virtual address `0x7fff1234` may actually be at physical address `0x2abc5678` in RAM. Other processes have completely different mappings. Each process is isolated — it cannot touch another's physical memory even if it tries.

**Why it matters:**

- **Isolation** — processes cannot corrupt each other's memory
- **More memory than physically available** — processes can use more virtual memory than there is RAM, using disk as overflow (swapping)
- **Shared libraries** — a shared library (like libc) can be mapped into every process's virtual address space, but only exists once in physical RAM

---

### Paging and Swapping

RAM is finite. When all RAM is used, the OS does not crash — it **pages out** (swaps) less recently used memory pages to disk, freeing RAM for active processes. When that memory is needed again, it is **paged back in** from disk.

**The cost:** Disk access is 1,000x slower than RAM. When a system starts swapping heavily, performance collapses. This is why production servers are sized to never use swap in normal operation.

> [!warning] Seen in Production A Java service with a memory leak gradually consumed more and more heap over hours. Eventually the OS started swapping. Response times went from 50ms to 30 seconds. The system appeared "up" but was functionally dead. The fix was finding the leak — but the lesson is: swapping is not a safety net. It is a symptom of a dying system.

---

### Memory Leaks

A **memory leak** occurs when a program allocates memory but never releases it. Over time, the process's memory footprint grows until the OS kills it — or the machine runs out of RAM entirely.

In garbage-collected languages (Java, Python, JavaScript), leaks occur when references are accidentally retained — an object is never used again, but something still holds a reference to it, so GC cannot collect it.

**Common causes in production:**

- Event listeners added but never removed
- Caches that grow unbounded (no eviction policy)
- Static collections that accumulate data forever
- Connection pools that never release unused connections

---

## 💽 B5 — Storage — Disk, Files, and I/O

Your database runs on a disk. Your logs are written to disk. Your backups live on disk. Understanding how storage works — and how different storage types behave — directly informs how you design databases, file systems, and cloud architectures.

---

### HDD vs SSD

**HDD — Hard Disk Drive** A mechanical device with spinning magnetic platters and a physical read/write head that moves to the correct position. The head physically moves — this mechanical seek takes **5–10 milliseconds**. On a modern server, this means an HDD can do roughly 100–200 random reads per second. Sequential reads are much faster — the head moves once and reads continuously.

**SSD — Solid State Drive** No moving parts. Data is stored in flash memory cells. Random reads take **~100 microseconds** — 100x faster than HDD. Sequential reads approach memory speeds.

||**HDD**|**SSD**|
|---|---|---|
|**Random read latency**|5–10 ms|0.1–0.2 ms|
|**Sequential read speed**|100–200 MB/s|500–7000 MB/s|
|**Cost per GB**|Very low|Low–medium|
|**Best for**|Cold storage, large archives|Databases, OS, active data|

---

### Sequential vs Random I/O

**Sequential I/O** — reading or writing data in order, from position A to position B in one continuous stream. Extremely fast on both HDD and SSD.

**Random I/O** — reading or writing data scattered across the disk in no particular order. Each read requires seeking to a new location. Fast on SSD. Very slow on HDD.

**Why databases care:** A database doing a full table scan reads data sequentially — fast. A database performing index lookups jumps around the disk randomly — on HDD, this is the bottleneck. Database design (indexes, storage layout, B-tree structures) is largely about converting random I/O into sequential I/O.

**Kafka** is designed almost entirely around sequential writes — it appends messages to log files in order, never doing random writes. This is how Kafka achieves extraordinary throughput even on spinning disks.

---

### File System Basics

The file system is the layer between your program and raw storage that organises data into files and directories.

**Inode** — every file in a Unix filesystem has an inode — a data structure containing all metadata about the file: owner, permissions, timestamps, size, and pointers to the actual data blocks on disk. The filename is stored separately — a directory is just a mapping from names to inode numbers.

**Why this matters:** When you move a file within the same filesystem, the OS just updates the directory entry — no data is copied. Instantaneous, regardless of file size. When you move a file across filesystems — data must be physically copied.

---

### Buffered vs Unbuffered I/O

When you write to a file, does the data go to disk immediately?

**Buffered I/O** (default): Data is written to an in-memory buffer first. The OS flushes the buffer to disk periodically or when it is full. Fast — most writes stay in RAM. Risk: if the machine crashes before the buffer is flushed, data is lost.

**Unbuffered / Direct I/O**: Data goes directly to the storage device with no OS buffering. Slower, but what you see is what is on disk.

**`fsync()`** — a system call that forces the OS to flush all buffered writes to disk immediately. Databases call `fsync()` after committing a transaction — ensuring the committed data is durable even if the machine crashes a millisecond later.

> [!tip] The Durability Tradeoff This is one of the most fundamental tradeoffs in storage systems. Fsync after every write = slow but durable. Buffer writes and flush periodically = fast but risk of data loss on crash. Different databases make different choices based on their consistency guarantees.

---

### The Three Storage Models in Cloud Systems

**Block Storage** Raw storage volumes — like a virtual hard disk. No file system — just addressable blocks of data. The application (or OS) manages the file system on top. Examples: AWS EBS, Azure Managed Disks. Used by: databases, VMs.

**File Storage** A file system accessible over a network. Multiple machines can mount the same file system simultaneously. Examples: AWS EFS, NFS, Azure Files. Used by: shared configuration, media files accessed by multiple servers.

**Object Storage** Store and retrieve unstructured data (files, images, videos, backups) by a unique key (URL). Infinitely scalable, highly durable (typically 99.999999999% — eleven nines). No hierarchy — just a flat namespace of key → value. Examples: AWS S3, Google Cloud Storage, Cloudflare R2. Used by: everything large and unstructured.

> [!info] Simple Rule Database? → Block storage. Shared config files between servers? → File storage. User profile pictures, videos, backups? → Object storage.

---

## ⚡ B6 — Caching at the OS Level

Before Redis, before CDNs, before application-level caches — caching exists at the hardware and OS level, automatically, invisibly. Understanding this is the mental foundation for understanding all caching, anywhere.

---

### CPU Cache — L1, L2, L3

When the CPU needs data, it looks in this exact order:

**L1 Cache** (per core, ~32KB, ~1ns) → **L2 Cache** (per core, ~256KB, ~4ns) → **L3 Cache** (shared, ~16MB, ~10ns) → **RAM** (~100ns) → **Disk** (~150,000ns)

The CPU hardware manages these caches automatically. Your code does not explicitly control them. But your code's data access patterns determine whether the CPU finds data in L1 (fast) or has to go all the way to RAM (100x slower).

**Cache line:** The CPU does not fetch individual bytes — it fetches 64-byte cache lines. If your data is laid out sequentially in memory, fetching one element brings nearby elements into cache too — ready for the next access.

---

### Page Cache

When your program reads a file, the OS copies it from disk into RAM — into a region called the **page cache**. If the same file is read again (by the same or a different process), the OS serves it from the page cache. No disk access needed.

This happens completely transparently. On a well-tuned Linux server, frequently accessed files may never touch the disk — they live in page cache permanently, served at RAM speed.

> [!example] Real Impact A fresh Linux server starts a database. First queries hit disk — 10ms each. After warm-up, the working set is cached in page cache. The same queries now take 0.1ms. 100x faster. Nothing changed in the database code — the OS cache did the work. This is why "warming up" a server after deployment matters.

---

### Cache Hit vs Cache Miss

**Cache hit** — the requested data is in cache. Served immediately at cache speed.

**Cache miss** — the data is not in cache. Must be fetched from the slower layer (disk, RAM, network). The data is then loaded into cache for future requests.

**Hit rate** — the percentage of requests served from cache. A 99% hit rate means 99 out of 100 reads never touch disk. Going from 95% to 99% hit rate cuts disk reads by 80%.

**The mental model to carry forward:** Every caching system you will ever design — Redis, Memcached, CDN, browser cache, HTTP cache headers — is the same idea as CPU cache and page cache, just at a different layer of the stack. Fast storage in front of slow storage. The principles never change.

---

## 🔁 B7 — Concurrency Patterns Used in System Design

These patterns appear everywhere — in message queues, databases, web servers, and distributed systems. They are the building blocks of concurrent system design.

---

### Producer-Consumer Pattern

**The idea:** One set of threads (producers) creates work items and puts them in a shared queue. Another set of threads (consumers) takes items from the queue and processes them. Producers and consumers are completely decoupled — they do not know about each other.

```
[Producer 1] ─┐
[Producer 2] ─┤──► [Queue] ──► [Consumer 1]
[Producer 3] ─┘               [Consumer 2]
```

**Why it is powerful:**

- Producers can generate work faster than consumers can process it — the queue absorbs the spike
- Consumers can be scaled independently of producers
- If consumers are slow, they do not block producers
- If producers are slow, consumers simply wait — no wasted spinning

**This is the exact model behind every message queue:** Kafka, RabbitMQ, AWS SQS — they are all industrial-strength implementations of the producer-consumer pattern. Your web server produces events (user actions, orders, notifications). Your backend workers consume and process them. The queue decouples the two.

---

### Reader-Writer Lock

**The problem:** A mutex allows only one thread at a time — even for reads. But reads do not conflict with each other. If 100 threads are reading a value, they can all safely do so simultaneously. The mutex forces them to wait in line unnecessarily.

**Reader-writer lock** separates the two cases:

- **Multiple readers simultaneously** — all allowed, no conflict
- **One writer at a time** — exclusive, blocks all readers and other writers

```
100 concurrent reads → all proceed simultaneously ✓
1 write → waits for all active reads to finish, then gets exclusive access ✓
```

**Where it is used:**

- Database engines — many transactions reading data simultaneously, occasional writes
- In-memory caches — many reads, occasional updates
- Configuration systems — many reads of config, rare updates

---

### Thread Pool Pattern

Already covered in B2 — a fixed set of pre-created threads processing a task queue. The most common concurrency pattern in production systems. Every web server, database, and executor uses this under the hood.

The key tuning decision: **how many threads in the pool?**

- Too few → requests queue up, latency increases
- Too many → excessive context switching, memory pressure

**Rule of thumb:**

- CPU-bound tasks → pool size ≈ number of CPU cores
- I/O-bound tasks → pool size can be much larger (threads spend time waiting, not computing)

---

### Async / Await and Futures

Modern languages provide language-level abstractions for async operations:

**Future / Promise** — a placeholder for a result that will be available later. Your code can continue executing while waiting for the future to resolve.

**Async / Await** — syntactic sugar that makes async code _look_ sequential, without the complexity of callbacks.

```
// Without async/await — callback hell
getUser(id, (user) => {
  getOrders(user, (orders) => {
    getItems(orders[0], (items) => { ... })
  })
})

// With async/await — reads like synchronous code
const user = await getUser(id)
const orders = await getOrders(user)
const items = await getItems(orders[0])
```

Same non-blocking behaviour. Dramatically more readable. Used in JavaScript, Python, Rust, C#, Swift, and most modern languages.

---

### Actor Model

**The problem with threads and shared memory:** as systems grow, managing who owns what data, which locks to acquire, and in what order becomes impossibly complex.

**The Actor Model** eliminates shared memory entirely. Each **actor** is an independent unit with:

- Its own private state — no other actor can touch it directly
- A mailbox — a queue of incoming messages
- Behaviour — how it reacts to each type of message

Actors communicate **only by sending messages**. No shared memory. No locks. No race conditions.

> [!example] Erlang and WhatsApp WhatsApp was built on Erlang, which is built around the Actor Model. Each WhatsApp connection, each chat, each user session is an actor. At peak, WhatsApp ran 2 million connected users per server — something traditional threaded servers could not approach. The Actor Model made massive concurrency manageable without explicit locking.

**Used in:** Erlang/Elixir, Akka (Java/Scala), Microsoft Orleans, many distributed system designs.

---

## 🖥️ B8 — OS Concepts That Show Up in System Design Interviews

---

### File Descriptors and Sockets

In Linux, **everything is a file**. Network connections, regular files, directories, devices, pipes — all are accessed through **file descriptors** — small integers that represent open resources.

When your server accepts a new connection, the OS assigns it a file descriptor. Reading from and writing to the network connection uses the same `read()` and `write()` system calls as reading from a file.

**A socket** is a special type of file descriptor — one that represents a network endpoint. Binding to a port, accepting connections, sending and receiving data — all done through socket file descriptors.

**Why this matters:** Every open file descriptor consumes OS resources. The default limit on many Linux systems is **1,024 file descriptors per process**. A server handling 1,000 simultaneous connections is at the limit. Tuning `ulimit -n` (the file descriptor limit) is one of the first things engineers do when setting up high-concurrency servers.

---

### Epoll — How Modern Servers Handle Thousands of Connections

**The naive approach:** For each connection, create a thread that calls `read()` and waits. With 10,000 connections, you have 10,000 waiting threads. This is the problem.

**The `select()` approach (old):** Give the OS a list of file descriptors to watch. The OS tells you which ones are ready. Better — but `select()` scans the entire list every time. With 10,000 file descriptors, that is 10,000 checks per event, per iteration.

**`epoll` (Linux) / `kqueue` (Mac/BSD) — the modern solution:** Register file descriptors with the OS once. The OS maintains an internal data structure. When any registered descriptor becomes ready (data arrives, connection accepted), the OS adds it to a ready list. `epoll_wait()` returns only the ready descriptors — not all 10,000, just the 3 that have activity.

**O(1) per event** regardless of total connections. This is the mechanism that allows Nginx to handle 50,000 simultaneous connections on a single thread. It is what makes the event loop possible at scale.

---

### The C10K Problem

In 1999, engineer Dan Kegel published a paper titled "The C10K Problem." The question: **how do you build a server that handles 10,000 simultaneous connections?**

At the time, the dominant model was one thread per connection. 10,000 connections = 10,000 threads. At 1MB of stack per thread, that is 10GB of RAM just for thread stacks — on servers with 512MB RAM. Even with less stack, the context switching overhead was crippling.

**The solutions that emerged:**

1. **Event-driven architecture** with epoll/kqueue — one thread, thousands of connections, OS notifies when ready
2. **Non-blocking I/O** — never block a thread waiting for network or disk
3. **Async processing** — callbacks, futures, event loops

These solutions became the foundation of Nginx, Node.js, Redis, and modern web infrastructure. The C10K problem was not just an academic exercise — solving it created the entire modern model of high-concurrency server design.

Today, the question is the **C10M problem** — 10 million simultaneous connections. Solutions involve kernel bypass (DPDK), zero-copy networking, and custom network stacks.

---

### System Calls — The Boundary Between Code and OS

Your application code cannot directly access hardware — disk, network, memory. It must ask the OS through **system calls** — a controlled interface into the kernel.

Common system calls:

- `open()`, `read()`, `write()`, `close()` — file I/O
- `socket()`, `bind()`, `listen()`, `accept()`, `connect()` — networking
- `malloc()` / `mmap()` — memory allocation
- `fork()`, `exec()` — process creation

System calls are **not free**. Each one requires switching from **user mode** to **kernel mode** — the CPU changes privilege level, context is saved and restored. This takes time — typically 1–5 microseconds.

Programs that make millions of system calls per second — like high-frequency database engines — spend significant time in mode switching. This is why performance-critical systems try to batch operations and minimise system call frequency (e.g. using large read/write buffers).

---

> [!success] Part B — Complete You now understand what happens inside the machines your systems run on. Processes and threads, concurrency and parallelism, locks and deadlocks, memory and storage, caching and I/O — these are the forces your code operates within. Every design decision in LLD and HLD is constrained and enabled by these fundamentals. You will reference this chapter constantly as we go deeper.

---

_Next →_ [[03-Low-Level-System-Design/LLD-Introduction|Chapter 3 — Low Level System Design]]

---

_— Sarvan Yaduvanshi · Competitive Programmer · LLM Foundation Model Engineer_