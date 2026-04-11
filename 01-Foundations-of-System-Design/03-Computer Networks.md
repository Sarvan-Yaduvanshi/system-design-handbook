# System Design — From Zero to Architect

_— Sarvan Yaduvanshi_

> Competitive Programmer · LLM Foundation Model Engineer · System Design Enthusiast

---

> [!quote] _"The internet is not a cloud. It is millions of machines, wires, and agreements — all working together so your message arrives in under a second."_

---

# Chapter 2 — CS Fundamentals for System Design

## Part A — Computer Networks

---

## 🌐 A1 — How the Internet Works

You have typed `https://google.com` thousands of times. But have you ever stopped and thought — what actually happens between pressing Enter and seeing that search page?

Most developers cannot answer this. But every system designer must be able to — because designing systems means understanding the very path your data travels.

Let us trace that journey, step by step.

---

### What is an IP Address?

Every device connected to the internet has an address. Just like your home has a postal address so the delivery person knows where to bring your package — every computer, phone, and server on the internet has an **IP Address** so data knows where to go.

**IPv4** — the original format. Looks like this: `142.250.190.46` Four numbers, each between 0 and 255, separated by dots. This gives us about **4.3 billion** possible addresses.

That sounds like a lot. But we ran out. The internet grew faster than anyone predicted. Every phone, laptop, smart TV, and IoT device needs an address. 4.3 billion was not enough.

**IPv6** — the solution. Looks like this: `2607:f8b0:4004:0c08:0000:0000:0000:200e` 128 bits instead of 32 bits. This gives us **340 undecillion** addresses — that is 340 followed by 36 zeros. We will not run out again.

> [!example] Real World Analogy IPv4 is like a city with 4.3 billion house numbers — already full. IPv6 is like redesigning the entire address system to give every grain of sand on Earth its own address — and still having room left over.

---

### What is a Port?

An IP address gets data to the right machine. But a machine runs dozens of services simultaneously — a web server, an email server, an SSH server, a database. How does the data know which service to go to?

That is what a **Port** is. A port is a number — 0 to 65535 — that identifies a specific service on a machine.

|Port|Service|
|---|---|
|**80**|HTTP — unencrypted web traffic|
|**443**|HTTPS — encrypted web traffic|
|**22**|SSH — secure shell, remote access|
|**21**|FTP — file transfer|
|**25**|SMTP — sending email|
|**5432**|PostgreSQL database|
|**3306**|MySQL database|
|**6379**|Redis cache|

So when your browser connects to `google.com`, it is actually connecting to `142.250.190.46:443` — Google's IP address, port 443, because HTTPS runs on port 443.

> [!tip] Think of It This Way The IP address is the building. The port is the flat number inside that building. The delivery (your data) goes to the right building AND the right flat.

---

### What is a Domain Name?

Humans cannot remember `142.250.190.46`. We remember `google.com`. The system that translates human-readable names into IP addresses is called **DNS** — the Domain Name System. We will cover DNS in full detail in section A4.

---

### What are Packets?

When you send a message or load a webpage, your data does not travel as one big block. It is broken into small pieces called **packets** — typically 1,500 bytes each.

Why? Because networks are shared. If you sent one enormous block of data, you would hog the entire network path. Other people's data would have to wait. By breaking data into small packets:

- Multiple people can share the same network path simultaneously
- If one packet gets lost, only that small piece needs to be resent — not everything
- Packets can take different routes and still arrive at the same destination

Each packet contains:

- A **header** — source IP, destination IP, packet number, total packets
- A **payload** — the actual data it is carrying

At the destination, all packets are reassembled in order. The webpage appears.

> [!example] Real Example Loading a 500KB webpage might generate ~340 packets. Each takes its own path across the internet — some through Mumbai, some through Singapore, some through Frankfurt. They all arrive at your browser within milliseconds and are reassembled into the page you see.

---

### Bandwidth vs Latency vs Throughput

These three terms are used constantly in system design. Most people confuse them.

**Latency** — the time it takes for one piece of data to travel from A to B. Measured in **milliseconds (ms)**. This is the _delay_.

_Example: Your request reaches Google's server in 12ms. That is latency._

**Bandwidth** — the maximum amount of data that can travel through a connection per second. Measured in **Mbps or Gbps**. This is the _capacity_ of the pipe.

_Example: Your home internet has 100 Mbps bandwidth._

**Throughput** — the actual amount of data that successfully travels per second, in practice. Always less than or equal to bandwidth. This is the _real-world speed_.

_Example: Your 100 Mbps connection actually delivers 78 Mbps because of overhead, packet loss, and congestion._

> [!tip] The Water Pipe Analogy **Bandwidth** = the diameter of the pipe — how much water _can_ flow. **Throughput** = how much water _actually_ flows right now. **Latency** = how long it takes for one drop of water to travel from one end to the other.
> 
> A wide pipe (high bandwidth) does not mean fast delivery (low latency). A fat pipe to another continent still has the speed of light as a limit.

**Why this matters in System Design:**

- A video streaming system optimises for **throughput** — you need continuous data flow
- A trading system optimises for **latency** — milliseconds mean money
- A file backup system optimises for **bandwidth** — move as much data as possible

---

### The Full Journey — `https://google.com`

Now let us put it all together.

**1.** You type `https://google.com` and press Enter.

**2.** Your browser checks its own cache — has it looked up `google.com` recently? If yes, it already knows the IP address. If no, it asks the operating system.

**3.** The OS checks its own DNS cache. If not found, it asks your configured DNS server — usually your router or your ISP's DNS server.

**4.** The DNS server resolves `google.com` to an IP address — say `142.250.190.46` — and returns it.

**5.** Your browser now connects to `142.250.190.46` on **port 443** (HTTPS). This initiates a **TCP connection** — a three-way handshake (we will cover this in A3).

**6.** Once connected, a **TLS handshake** happens — your browser and Google's server agree on encryption keys so the conversation is private (covered in A6).

**7.** Your browser sends an **HTTP GET request** over this encrypted connection — _"Give me the homepage of google.com"_ (covered in A5).

**8.** Google's server processes the request and sends back the HTML, CSS, and JavaScript — broken into packets.

**9.** Your browser receives all the packets, reassembles them, parses the HTML, and renders the page.

**Total time: Under 200 milliseconds.**

> [!info] Every Step Has Depth Each of the steps above — DNS, TCP, TLS, HTTP — has its own entire section in this chapter. By the end of Part A, you will be able to explain every millisecond of that journey in detail.

---

## 🔗 A2 — The OSI Model

When engineers from different companies build networking hardware and software, they need a common language — a shared agreement about how data should move from one machine to another. That agreement is the **OSI Model**.

OSI stands for **Open Systems Interconnection**. It divides the complex problem of networking into **seven distinct layers**, each with a specific responsibility.

Think of it like a factory assembly line. Each layer adds something to the product before passing it to the next layer.

---

### The Seven Layers — Top to Bottom

When you send data, it travels **down** the layers on your device, across the network, then **up** the layers on the receiving device.

---

#### Layer 7 — Application Layer

**What it does:** The layer your applications actually interact with. This is where protocols like HTTP, HTTPS, DNS, SMTP, and FTP live.

**Your code lives here.** When you build an API or a web server, you are working at Layer 7.

_Examples: Chrome browser, Postman, your Node.js server, email clients._

---

#### Layer 6 — Presentation Layer

**What it does:** Translates data into a format both sides can understand. Handles encryption, decryption, encoding, and compression.

When you send JSON from your browser to a server — formatting that JSON, compressing it, and encrypting it happens at this layer.

_Examples: SSL/TLS encryption, JPEG compression, UTF-8 encoding, JSON serialisation._

---

#### Layer 5 — Session Layer

**What it does:** Manages the opening, maintaining, and closing of communication sessions between two devices. Keeps track of which conversation belongs to which connection.

When you log into a website and your session stays active across multiple requests — that session management is Layer 5 thinking.

_Examples: Session tokens, API authentication sessions, RPC session management._

---

#### Layer 4 — Transport Layer

**What it does:** Responsible for end-to-end delivery of data between two applications. Decides whether delivery should be **reliable (TCP)** or **fast (UDP)**. Handles segmentation, flow control, and error recovery.

**This is one of the most important layers for System Design.** The choice between TCP and UDP is a Layer 4 decision and it appears in almost every System Design interview.

_Examples: TCP, UDP, port numbers._

---

#### Layer 3 — Network Layer

**What it does:** Responsible for routing packets from the source machine to the destination machine across multiple networks. This is where **IP addresses** live and where routers operate.

When your packet leaves your home and travels through your ISP, then through several routers, then reaches Google's data center — that routing is Layer 3.

_Examples: IP (IPv4, IPv6), routers, routing tables, ICMP (the `ping` command)._

---

#### Layer 2 — Data Link Layer

**What it does:** Responsible for transferring data between two devices on the **same network** — for example, your laptop and your router. Uses **MAC addresses** (not IP addresses) to identify devices locally.

_Examples: Ethernet, Wi-Fi (802.11), MAC addresses, switches, ARP (Address Resolution Protocol)._

---

#### Layer 1 — Physical Layer

**What it does:** The raw physical transmission of bits — 0s and 1s — over a medium. This layer is about cables, radio waves, light pulses, and electrical signals.

_Examples: Ethernet cables, fibre optic cables, Wi-Fi radio signals, Bluetooth._

---

### The OSI Model — Summary Table

|Layer|Name|Key Responsibility|Examples|
|---|---|---|---|
|7|Application|What apps see|HTTP, HTTPS, DNS, FTP|
|6|Presentation|Format, encrypt, compress|TLS, JSON, JPEG|
|5|Session|Manage sessions|Sessions, Auth tokens|
|4|Transport|Reliable or fast delivery|TCP, UDP, Ports|
|3|Network|Routing across networks|IP, Routers|
|2|Data Link|Local delivery|MAC, Ethernet, Switches|
|1|Physical|Raw bits on a medium|Cables, Radio, Fibre|

---

### Which Layers Matter Most for System Design?

**Layer 7 (Application)** — You work here every day. APIs, HTTP, DNS decisions.

**Layer 4 (Transport)** — TCP vs UDP is a critical design decision for real-time systems, streaming, and gaming.

**Layer 3 (Network)** — Understanding IP routing matters when you design multi-region systems, VPCs, and private networks in the cloud.

**Layers 1, 2** — You can treat these as a black box for most System Design work. They are more relevant for network engineers than software architects.

> [!tip] The Mnemonic A common way to remember the layers top to bottom: **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing Application → Presentation → Session → Transport → Network → Data Link → Physical

---

## 🔄 A3 — TCP vs UDP

Every piece of data that travels across the internet uses one of two transport protocols — **TCP** or **UDP**. The choice between them is one of the most fundamental decisions in System Design.

---

### TCP — Transmission Control Protocol

TCP is the **reliable** option. When you send data over TCP, you get a guarantee — the data will arrive, it will arrive in the correct order, and if anything gets lost, it will be resent automatically.

How does TCP achieve this reliability? Through a process called the **Three-Way Handshake**.

---

#### The Three-Way Handshake

Before a single byte of real data is sent, TCP makes both sides agree that they are ready to communicate. This happens in three steps:

**Step 1 — SYN (Synchronise)** Your computer sends a SYN packet to the server. _"Hey, I want to start a conversation. Are you there?"_

**Step 2 — SYN-ACK (Synchronise-Acknowledge)** The server receives the SYN and responds with SYN-ACK. _"Yes, I am here. I am ready. Are you still there?"_

**Step 3 — ACK (Acknowledge)** Your computer responds with ACK. _"Yes, I am ready. Let us begin."_

Only after this handshake does actual data start flowing.

> [!example] Real Life Analogy Imagine calling someone on the phone before a meeting. You: _"Hello, can you hear me?"_ (SYN) Them: _"Yes, I can hear you. Can you hear me?"_ (SYN-ACK) You: _"Yes, perfectly. Let us start."_ (ACK) Now the actual conversation begins.

---

#### What TCP Guarantees

- **Delivery** — if a packet is lost, TCP detects it and resends it
- **Order** — packets are numbered. Even if they arrive out of order, TCP reassembles them correctly
- **Error Checking** — each packet has a checksum. Corrupted packets are discarded and resent
- **Flow Control** — TCP slows down the sender if the receiver is overwhelmed
- **Congestion Control** — TCP detects network congestion and reduces its speed to avoid making it worse

**Where TCP is used:**

- HTTP and HTTPS — every webpage, every API call
- Email — SMTP, IMAP, POP3
- File transfers — FTP, SFTP
- SSH — secure remote connections
- Database connections — MySQL, PostgreSQL, MongoDB

---

### UDP — User Datagram Protocol

UDP is the **fast** option. When you send data over UDP, you get no guarantees. The data might arrive. It might not. It might arrive out of order. UDP does not know and does not care.

That sounds terrible. Why would anyone use this?

Because **speed matters more than reliability** in many situations.

UDP has no handshake. No retransmission. No acknowledgement. You just fire packets into the network and move on. This makes UDP significantly faster and uses far less overhead.

---

#### Where UDP is Used and Why

**Video streaming (YouTube, Netflix)** If one frame of video is lost, there is no point in pausing the entire video to wait for it to be resent. By the time it arrives, that moment has passed. Better to just drop it and show the next frame. A tiny glitch is better than a pause.

**Live gaming (Call of Duty, PUBG, Valorant)** In a live multiplayer game, you need to know where your opponent is _right now_ — not where they were 200ms ago when a retransmission finally arrives. Speed is everything. A dropped position update is forgotten instantly.

**VoIP and video calls (Zoom, Google Meet, WhatsApp calls)** Same principle as gaming. A tiny audio glitch is acceptable. Freezing for a second while waiting for a retransmitted packet is not.

**DNS lookups** DNS queries are tiny — a few bytes. They are sent over UDP because it is fast, and if the response does not come back quickly, the client simply asks again. No need for the overhead of TCP.

**Live sports scores and stock price tickers** Data is outdated the moment it is generated. Getting the latest update fast beats waiting for a guaranteed delivery of an old update.

---

### TCP vs UDP — The Comparison Table

||**TCP**|**UDP**|
|---|---|---|
|**Connection**|Connection-based (handshake required)|Connectionless (fire and forget)|
|**Reliability**|Guaranteed delivery|No guarantee|
|**Order**|Guaranteed ordering|No ordering|
|**Speed**|Slower (overhead of reliability)|Faster (no overhead)|
|**Error Recovery**|Yes — retransmits lost packets|No — lost packets are gone|
|**Use When**|Data accuracy is critical|Speed is critical, some loss is acceptable|
|**Examples**|HTTP, email, file transfer, SSH|Video streaming, gaming, DNS, VoIP|

> [!tip] The Interview Answer If you are asked "why does your system use UDP?" — the answer is always some version of: _"Because the cost of retransmission (latency, pausing) is worse than the cost of occasional data loss, and the use case can tolerate imperfect delivery."_

---

## 🗂️ A4 — DNS — Domain Name System

Every time you type a website name into your browser, before any webpage can load, your computer needs to find out the IP address of the server hosting that site. The system that does this is **DNS — the Domain Name System**.

DNS is, at its core, a **massive distributed database** that maps human-readable names like `youtube.com` to machine-readable IP addresses like `142.250.195.14`.

It is one of the oldest and most critical systems on the internet — and it is itself one of the most elegant distributed system designs ever built.

---

### Why DNS Exists

In the early days of the internet, there were so few computers that a single file — called `hosts.txt` — listed every computer's name and IP address. Every computer had a copy. When a new machine joined, you updated the file and distributed it.

That worked for dozens of machines. It collapsed completely at thousands.

DNS replaced the hosts file with a distributed, hierarchical, cached, globally replicated database that handles **billions of lookups per day** with sub-millisecond response times.

---

### How DNS Resolution Works — Step by Step

You type `api.github.com` in your browser. Here is exactly what happens:

**Step 1 — Browser Cache** Your browser first checks its own DNS cache. If it looked up `api.github.com` recently and the result has not expired, it uses the cached IP address immediately. No network request needed.

**Step 2 — OS Cache** If the browser has no cached result, it asks the operating system. The OS has its own DNS cache. On Mac and Linux, you can inspect this. On most systems, the `/etc/hosts` file is checked first — this is the modern descendant of that original hosts.txt.

**Step 3 — Recursive Resolver** If the OS has no answer, it asks a **Recursive Resolver** — usually your ISP's DNS server, or a public one like Google's `8.8.8.8` or Cloudflare's `1.1.1.1`. This resolver does the hard work of finding the answer.

**Step 4 — Root Name Server** The recursive resolver asks a **Root Name Server** — _"Who knows about `.com` domains?"_ There are only 13 sets of root name servers in the world, operated by organisations like ICANN, Verisign, and NASA. They do not know the IP addresses — they only know who to ask next.

**Step 5 — TLD Name Server** The root server says — _"Ask the .com TLD (Top Level Domain) name server."_ The TLD server knows who is responsible for `github.com` specifically.

**Step 6 — Authoritative Name Server** The TLD server says — _"GitHub's own name server is responsible for `github.com`."_ The recursive resolver asks GitHub's authoritative name server, which returns the actual IP address for `api.github.com`.

**Step 7 — The Answer Returns** The IP address travels back to the recursive resolver, then to your OS, then to your browser. The browser caches it. The connection to `api.github.com` begins.

**This entire process takes 20–120ms. After that, the cached result is used instantly.**

---

### DNS Record Types

DNS stores different types of records for different purposes:

|Record|Full Name|What It Does|Example|
|---|---|---|---|
|**A**|Address|Maps a domain to an IPv4 address|`github.com → 140.82.121.4`|
|**AAAA**|IPv6 Address|Maps a domain to an IPv6 address|`github.com → 2606:50c0:...`|
|**CNAME**|Canonical Name|Maps a domain to another domain name|`www.github.com → github.com`|
|**MX**|Mail Exchange|Specifies the mail server for a domain|`github.com → mail.github.com`|
|**TXT**|Text|Stores arbitrary text — used for verification|SPF, DKIM email security records|
|**NS**|Name Server|Specifies which servers are authoritative for a domain|`github.com → ns1.p16.dynect.net`|

> [!example] Real Example — CNAME in Action When you deploy a website on Vercel, they give you a long URL like `my-app.vercel.app`. You want your users to visit `mysite.com` instead. You create a CNAME record in your DNS: `mysite.com → my-app.vercel.app`. Now DNS resolves `mysite.com` → looks up `my-app.vercel.app` → gets the real IP. Your custom domain works — no code change required.

---

### TTL — Time to Live

Every DNS record has a **TTL** — a number of seconds that tells resolvers how long to cache the result before asking again.

`A github.com TTL 60` — means: cache this for 60 seconds, then ask again.

**High TTL (3600 seconds — 1 hour):**

- Fewer DNS lookups → faster for users → less load on your DNS servers
- But — if you change your IP address (moving servers), the old IP stays cached for up to an hour everywhere in the world. Users hit the wrong server.

**Low TTL (60 seconds):**

- Changes propagate quickly — within a minute, the world sees your new IP
- But — more DNS lookups → slightly more overhead

> [!warning] Incident Story In 2021, a major cloud provider changed DNS records but forgot they had set a 24-hour TTL. Even after pointing DNS to the new servers, millions of users around the world continued hitting the old, now-offline servers for up to 24 hours. The fix was deployed. The outage continued. There was nothing to do but wait for TTLs to expire. Always lower your TTL before a planned migration.

---

### How System Designers Use DNS

**DNS Load Balancing** An A record can return multiple IP addresses for the same domain. DNS resolvers rotate through them — round robin. Requests get distributed across servers. Simple, global, requires no dedicated load balancer.

**Geographic Routing** A DNS server can return different IP addresses based on where the request comes from. A user in India gets routed to a server in Mumbai. A user in Germany gets routed to a server in Frankfurt. This reduces latency for users worldwide. Services like AWS Route 53 and Cloudflare do this automatically.

**Failover** DNS health checks monitor your servers. If the primary server goes down, DNS automatically stops returning that IP and returns the backup IP instead. Failover can happen in seconds.

---

## 🌐 A5 — HTTP and HTTPS

HTTP is the **language that the web speaks**. Every time your browser loads a page, every time your app calls an API, every time data moves between a client and a server on the web — it almost certainly uses HTTP.

Understanding HTTP deeply is not optional for a system designer. It is foundational.

---

### What is HTTP?

**HTTP — HyperText Transfer Protocol** — is a request-response protocol. A client sends a request. A server sends a response. That is the entire model.

It is **stateless** — each request is completely independent. The server does not remember the previous request. Every request must carry all the information the server needs to respond correctly.

---

### Anatomy of an HTTP Request

Every HTTP request has four parts:

**1. Method** — what action is being requested **2. URL** — where the request is going **3. Headers** — metadata about the request **4. Body** — optional data being sent with the request

Here is a real example — calling a GitHub API:

```
GET /users/sarvan/repos HTTP/1.1
Host: api.github.com
Authorization: Bearer ghp_xxxxxxxxxxxx
Accept: application/json
Content-Type: application/json
```

- `GET` — the method — we are requesting data, not sending any
- `/users/sarvan/repos` — the path on the server
- `HTTP/1.1` — the version of HTTP being used
- `Host`, `Authorization`, `Accept` — headers providing metadata

---

### HTTP Methods — What Each One Means

|Method|Purpose|Has Body?|Idempotent?|
|---|---|---|---|
|**GET**|Retrieve data|No|Yes|
|**POST**|Create new resource|Yes|No|
|**PUT**|Replace entire resource|Yes|Yes|
|**PATCH**|Update part of a resource|Yes|No|
|**DELETE**|Remove a resource|No|Yes|
|**OPTIONS**|Ask what methods are allowed|No|Yes|
|**HEAD**|Like GET, but returns only headers, no body|No|Yes|

**Idempotent** means: making the same request multiple times produces the same result. GET is idempotent — asking for the same data 10 times gives you the same answer. POST is not — creating a user 10 times creates 10 users.

> [!example] Real-World Mapping You are building a blog API: `GET /posts` → get all blog posts `GET /posts/42` → get post with ID 42 `POST /posts` → create a new post `PUT /posts/42` → replace post 42 entirely `PATCH /posts/42` → update just the title of post 42 `DELETE /posts/42` → delete post 42

---

### HTTP Status Codes — The Language of Responses

Every HTTP response comes with a three-digit status code that tells the client what happened.

**1xx — Informational** The request is in progress. Rarely seen in application development. `100 Continue` — keep sending your request body

**2xx — Success** The request worked. `200 OK` — standard success `201 Created` — a new resource was created (response to POST) `204 No Content` — success, but nothing to return (common for DELETE)

**3xx — Redirection** The client needs to go somewhere else. `301 Moved Permanently` — the resource has moved forever, update your bookmarks `302 Found` — temporary redirect, keep using the old URL next time `304 Not Modified` — your cached version is still valid, no need to re-download

**4xx — Client Error** The client did something wrong. `400 Bad Request` — your request is malformed `401 Unauthorized` — you need to log in first `403 Forbidden` — you are logged in, but you do not have permission `404 Not Found` — the resource does not exist `429 Too Many Requests` — you have been rate limited

**5xx — Server Error** The server did something wrong. `500 Internal Server Error` — something crashed on the server `502 Bad Gateway` — the server got an invalid response from an upstream service `503 Service Unavailable` — the server is overloaded or down for maintenance `504 Gateway Timeout` — the upstream service took too long to respond

> [!tip] For System Designers Status codes are not just for browsers. When you design APIs, choosing the right status code communicates meaning to every client. A `404` means "this never existed." A `410 Gone` means "this existed but was deleted." The distinction matters for SEO, for clients that cache responses, and for debugging.

---

### HTTP/1.1 vs HTTP/2 vs HTTP/3

HTTP has evolved significantly. Each version solved real performance problems.

**HTTP/1.1 — 1997** The standard for over 15 years. Reliable but has limitations:

- One request at a time per connection. To load faster, browsers opened 6 parallel connections per domain — a workaround, not a solution.
- Headers are sent as plain text on every request — even the same headers repeated thousands of times.
- **Head-of-line blocking** — if one request stalls, everything behind it waits.

**HTTP/2 — 2015** A complete reimagining of the transport mechanism, while keeping the same semantics.

- **Multiplexing** — multiple requests and responses fly over a single connection simultaneously. No more waiting in line.
- **Header compression (HPACK)** — repeated headers are compressed. Significant savings for API-heavy applications.
- **Server Push** — the server can proactively send resources the client will need before being asked. A browser requests `index.html`; the server immediately sends `style.css` and `script.js` too.
- **Binary protocol** — faster for machines to parse than HTTP/1.1's text format.

**HTTP/3 — 2022** Built on **QUIC** instead of TCP. This is the big change.

- QUIC runs on **UDP** with reliability built on top — getting the best of both worlds.
- Eliminates head-of-line blocking at the transport layer — a lost packet in one stream does not stall other streams.
- **Faster connection setup** — combines the TCP and TLS handshakes into one. Connections are established faster, especially on mobile networks where users switch between Wi-Fi and cellular.
- Already used by Google, YouTube, and Meta for a significant portion of their traffic.

---

### Cookies, Sessions, and Headers — Carrying State in a Stateless Protocol

HTTP is stateless. The server remembers nothing between requests. But users need to stay logged in, shopping carts need to persist, and preferences need to be saved. How?

**Cookies** Small pieces of data the server sends to the browser with a `Set-Cookie` header. The browser stores them and automatically includes them in every subsequent request to that domain.

```
Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Strict
```

- `HttpOnly` — JavaScript cannot read this cookie. Prevents XSS attacks.
- `Secure` — only sent over HTTPS. Never over plain HTTP.
- `SameSite=Strict` — only sent with requests originating from the same site. Prevents CSRF attacks.

**Sessions** The server creates a session object (stored in memory, Redis, or a database) and sends the session ID to the client as a cookie. On every request, the client sends the session ID back. The server looks it up and knows who you are.

**JWT — JSON Web Tokens** An alternative to sessions. Instead of storing session state on the server, the entire user identity is encoded into a signed token that the client stores. The server verifies the signature — no database lookup needed. Scales much better but comes with tradeoffs (tokens cannot be revoked easily until they expire).

---

## 🔒 A6 — TLS and SSL — How Encryption Works

When you connect to `https://` anything, your data is encrypted. Even if someone intercepts your packets between your laptop and the server — they see nothing but meaningless gibberish. This is the work of **TLS**.

Understanding TLS is essential for system designers because decisions about where TLS terminates, how certificates are managed, and the performance cost of encryption affect every production system.

---

### SSL vs TLS — What is the Difference?

**SSL (Secure Sockets Layer)** was the original protocol, developed by Netscape in 1995. It has been deprecated and broken. SSL 2.0 and SSL 3.0 are considered insecure and should not be used anywhere.

**TLS (Transport Layer Security)** is the modern, secure replacement. Current version is **TLS 1.3** (2018).

> [!warning] Common Confusion When people say "SSL certificate" or "SSL encryption," they almost always mean TLS. The term "SSL" stuck in common usage even though TLS replaced it. In technical conversations, say TLS. In customer conversations, SSL is still widely understood and acceptable.

---

### Symmetric vs Asymmetric Encryption

To understand TLS, you first need to understand two types of encryption.

**Symmetric Encryption** One key. The same key encrypts the data and decrypts it. Fast and efficient. The problem: how do you share that key with the other person without someone intercepting it?

_Analogy: A padlock where you and your friend both have a copy of the same key. But how do you get your friend a copy of the key without someone stealing it in transit?_

**Asymmetric Encryption** Two keys — a **public key** and a **private key**. They are mathematically linked. Data encrypted with the public key can only be decrypted with the private key, and vice versa.

You can share your public key with anyone. Everyone can see it. But only you have the private key. Someone encrypts a message with your public key — only you can read it with your private key.

_Analogy: A special padlock you can share with anyone open. Anyone can click it shut (encrypt with your public key). But only you have the key to open it (private key)._

**Asymmetric encryption is slow.** It requires complex mathematics. You cannot use it to encrypt a 500MB file.

**TLS uses both** — asymmetric encryption to securely exchange a symmetric key, then uses that symmetric key for the actual data transfer. Best of both worlds.

---

### The TLS Handshake — Step by Step

**Step 1 — Client Hello** Your browser sends: _"Hello. I support these TLS versions and these encryption algorithms. Here is a random number."_

**Step 2 — Server Hello** The server responds: _"Hello. I choose TLS 1.3. I choose this cipher suite. Here is my certificate. Here is my random number."_

**Step 3 — Certificate Verification** Your browser examines the server's certificate. A certificate contains the server's public key and is signed by a **Certificate Authority (CA)** — a trusted third party like DigiCert, Let's Encrypt, or Comodo.

Your browser has a built-in list of trusted CAs. It checks: did a trusted CA sign this certificate? Is it expired? Is it for the correct domain? If yes — the server is who it claims to be.

**Step 4 — Key Exchange** Using the server's public key and both random numbers, both sides independently compute the same **session key** — a symmetric key that will encrypt all data for this session. The mathematics ensure that even if the entire conversation was recorded, the session key cannot be derived from the intercepted data.

**Step 5 — Encrypted Communication Begins** Both sides confirm they are ready. All subsequent communication is encrypted with the session key. An interceptor sees only random bytes.

---

### TLS Termination in System Design

In a real production system, HTTPS traffic often does not travel all the way to your application server encrypted. It is decrypted earlier — at a **load balancer**, **reverse proxy**, or **API gateway**. This is called **TLS termination**.

```
User (HTTPS) → Load Balancer (TLS terminates here) → Backend servers (plain HTTP)
```

**Why?**

- Your backend servers do not need to do the computational work of encryption/decryption
- The load balancer handles certificates centrally — you do not need to install certs on every server
- Internal traffic (within a private data center or VPC) is often trusted and does not need encryption

**The tradeoff:** Internal traffic between your load balancer and backend servers travels unencrypted. This is acceptable only if your internal network is genuinely private and secured. For highly sensitive systems, end-to-end encryption (E2EE) is used instead.

---

## ⚡ A7 — WebSockets and Real-Time Communication

HTTP is a request-response protocol. The client asks, the server answers, the conversation is over. This works perfectly for loading webpages and calling APIs.

But what about **real-time systems** — systems where the server needs to push data to the client continuously, without the client asking?

- A WhatsApp message arriving on your screen
- A live cricket score updating every ball
- A stock price changing every second
- A collaborative Google Doc where you see your colleague's cursor moving in real time
- A multiplayer game where every player's position updates 60 times per second

HTTP alone cannot handle these efficiently. This section covers the four approaches — from naive to optimal.

---

### Approach 1 — Short Polling

**The idea:** The client asks the server every N seconds — _"Is there anything new?"_

```
Client: "Any new messages?" → Server: "No."   (t=0s)
Client: "Any new messages?" → Server: "No."   (t=1s)
Client: "Any new messages?" → Server: "No."   (t=2s)
Client: "Any new messages?" → Server: "Yes! Here they are."  (t=3s)
```

**The problem:**

- Wasteful — most responses are empty
- High latency — a message arrives 0.5 seconds after your last poll but you will not see it for almost 1 full second
- At scale, millions of clients polling every second creates enormous server load

**When it is acceptable:** Very low-frequency updates. Checking email every 5 minutes. Refreshing a leaderboard every 30 seconds.

---

### Approach 2 — Long Polling

**The idea:** The client sends a request, but the server _holds it open_ — does not respond until it has something new. As soon as data is available, the server responds. The client immediately sends another request and waits again.

```
Client: "Any new messages?" → Server holds the connection open...
Server: (30 seconds later) "Yes! Here they are." → Client immediately asks again
Client: "Any new messages?" → Server holds...
```

**Improvement over short polling:**

- Much lower latency — data is pushed the instant it is available
- Fewer wasted responses

**The problem:**

- Still creates a new HTTP connection for each update
- Server holds thousands of open connections — resource intensive
- Complex to implement correctly — timeouts, reconnection logic

**When it is used:** Some legacy chat systems, certain notification systems.

---

### Approach 3 — Server-Sent Events (SSE)

**The idea:** The client opens a single HTTP connection to the server. The server then streams events down to the client continuously — one-way, server to client only.

```
Client opens connection: GET /events
Server: (streams continuously)
  "data: Score is 45/2\n\n"
  "data: Score is 52/2\n\n"
  "data: Score is 52/3\n\n"
```

**Advantages:**

- Single persistent connection — efficient
- Built into browsers natively with the `EventSource` API
- Automatic reconnection built in
- Works over standard HTTP/2

**The limitation:** One-way only. The client cannot send data back through the same connection. If you only need server-to-client updates — SSE is excellent.

**Real examples:** Live sports scores, news feeds, real-time analytics dashboards, stock tickers.

---

### Approach 4 — WebSockets

**The idea:** A completely different protocol — a full-duplex, persistent, bidirectional connection between client and server. Both sides can send data to each other at any time, simultaneously, with minimal overhead.

---

#### The WebSocket Handshake

A WebSocket connection begins as an HTTP request — then upgrades:

```
GET /chat HTTP/1.1
Host: chat.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
```

The server responds:

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
```

After this, the HTTP connection is replaced by the WebSocket protocol. A raw, persistent, bidirectional channel exists. Both sides can send **frames** — small chunks of data — instantly, with almost zero overhead.

---

#### Why WebSockets Are Powerful

- **Truly bidirectional** — the client can send messages AND receive them over the same connection
- **Low latency** — no handshake overhead after the initial connection. Sending a message takes microseconds
- **Low overhead** — WebSocket frames add only 2-10 bytes of overhead. Compare to HTTP requests which add hundreds of bytes of headers every time
- **Push without polling** — the server pushes data the instant it is ready

---

### Comparison Table

||**Short Polling**|**Long Polling**|**SSE**|**WebSockets**|
|---|---|---|---|---|
|**Direction**|Client → Server|Client → Server|Server → Client|Both ways|
|**Latency**|High|Medium|Low|Very Low|
|**Connections**|Many (one per poll)|One at a time|One persistent|One persistent|
|**Complexity**|Simple|Medium|Simple|Medium|
|**Use When**|Rare updates|Moderate updates|Server pushes only|Full real-time, bidirectional|

**Real examples of WebSockets:**

- WhatsApp Web — messages appear instantly, both sides send and receive
- Google Docs — live cursor positions, edits appear as others type
- Zerodha Kite — live stock prices updating every second
- Multiplayer games — player positions, game state synchronized continuously
- Slack — real-time message delivery and typing indicators

---

## 🔌 A8 — APIs — The Contracts Between Systems

An **API (Application Programming Interface)** is a defined contract — a set of rules that says: _"If you send me this, I will send you that."_

APIs are what allow systems to talk to each other. Your frontend talks to your backend through an API. Your backend talks to payment services, email providers, and databases through APIs. Modern software is built by connecting APIs.

There are several API styles. Choosing the right one is a key system design decision.

---

### REST — Representational State Transfer

REST is the dominant API style of the web. It is not a protocol — it is a set of architectural principles, defined by Roy Fielding in his 2000 doctoral dissertation.

**The core principles of REST:**

**1. Stateless** Every request must contain all information needed to process it. The server stores no client state between requests. If you need authentication, send the token with every request. This allows any server to handle any request — critical for horizontal scaling.

**2. Resource-Based** REST thinks in terms of **resources** — entities that have identities. A user is a resource. A blog post is a resource. An order is a resource.

Each resource is identified by a URL:

- `/users` — the collection of all users
- `/users/42` — the specific user with ID 42
- `/users/42/orders` — all orders belonging to user 42

**3. Uniform Interface** HTTP methods define the operation. The URL defines the resource. Together they express any CRUD operation.

**REST vs CRUD — they are not the same thing**

|Operation|REST|HTTP Method|
|---|---|---|
|Create a user|`POST /users`|POST|
|Read a user|`GET /users/42`|GET|
|Update a user|`PUT /users/42`|PUT|
|Partially update|`PATCH /users/42`|PATCH|
|Delete a user|`DELETE /users/42`|DELETE|

---

### GraphQL

GraphQL is a **query language for APIs**, developed by Facebook in 2012 and open-sourced in 2015. It solves a specific problem that REST struggles with: **over-fetching and under-fetching**.

**The problem with REST:** Imagine a mobile app showing a user's profile — name, avatar, and their last 3 posts. In REST:

- `GET /users/42` → returns name, avatar, email, phone, address, preferences... much more than needed
- `GET /users/42/posts?limit=3` → second request needed

You either get too much data (over-fetching) or you need multiple requests (under-fetching).

**GraphQL's solution:** The client specifies _exactly_ what data it needs:

```graphql
query {
  user(id: 42) {
    name
    avatar
    posts(limit: 3) {
      title
      publishedAt
    }
  }
}
```

One request. Exactly the data requested. Nothing more.

**The N+1 Problem and how GraphQL solves it:** In REST, fetching 10 posts and their authors might mean 1 request for posts + 10 separate requests for each author — 11 total. This is the N+1 problem. GraphQL resolvers can batch these using a technique called **DataLoader** — one query for all 10 authors simultaneously.

**When to choose GraphQL:**

- Mobile applications where bandwidth and request count matter
- When different clients (mobile, web, third-party) need different shapes of data
- Complex data relationships with many entity types

---

### gRPC — Google Remote Procedure Call

gRPC is a high-performance API framework developed by Google. Where REST is text-based and human-readable, gRPC is **binary and machine-optimised**.

**Protocol Buffers (Protobuf)** Instead of JSON (which is text), gRPC uses Protobuf — a binary serialisation format. You define your data structure in a `.proto` file:

```protobuf
message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
}
```

Protobuf serialised data is **3-10x smaller** than equivalent JSON and **5-10x faster** to serialise/deserialise. For high-throughput internal services handling millions of requests per second, this matters enormously.

**Why gRPC is used for microservice communication:**

- Performance — significantly faster than REST with JSON
- Strict contracts — the `.proto` file is the exact contract between services. Type safety across service boundaries.
- Bidirectional streaming — gRPC supports streaming in both directions natively, built on HTTP/2
- Code generation — from a `.proto` file, gRPC generates client and server code in 10+ languages

**gRPC vs REST:**

- REST wins for public APIs — human-readable, easy to test with `curl`, every language supports it
- gRPC wins for internal microservice-to-microservice communication — performance, strict typing, streaming

---

### Webhooks — Reverse APIs

Traditional APIs are **pull** — you ask, you receive. Webhooks are **push** — when something happens, the server tells you.

**Example:** You are building a payment integration with Razorpay. When a payment succeeds, Razorpay needs to tell your system. You could poll `GET /payments/status` every second — wasteful. Instead, you register a webhook URL with Razorpay: `https://yourapp.com/webhooks/payment`. When a payment succeeds, Razorpay makes a `POST` request to your URL with the payment details. Your system receives it instantly.

**Webhooks are used by:**

- Payment gateways — Stripe, Razorpay, PayU
- Git platforms — GitHub, GitLab (triggering CI/CD pipelines)
- Communication tools — Slack, Twilio (incoming message notifications)
- E-commerce — Shopify order events

---

### API Versioning

APIs evolve. New fields are added, old ones are deprecated, endpoints change. But existing clients that depend on the old API must not break.

**URL Versioning (most common):** `/api/v1/users` and `/api/v2/users` coexist. Old clients keep using v1. New clients use v2.

**Header Versioning:** `Accept: application/vnd.myapp.v2+json` — the version is in the request header.

**Query Parameter Versioning:** `/api/users?version=2`

**Best practice:** URL versioning is the most explicit and easiest to debug. Most large APIs (Stripe, Twitter, Google) use URL versioning.

---

## 🔀 A9 — Proxies and Reverse Proxies

A **proxy** is an intermediary — a middleman that sits between two parties and handles communication on their behalf. Understanding proxies is essential because they appear everywhere in production systems — in front of clients, in front of servers, handling security, performance, and routing.

---

### Forward Proxy — Hiding the Client

A **forward proxy** sits between the **client and the internet**. The client sends requests to the proxy, and the proxy forwards them to the destination on the client's behalf. The destination server sees the proxy's IP address — not the client's.

```
Client → Forward Proxy → Internet → Server
```

**Use cases:**

**Corporate firewalls and content filtering** A company routes all employee internet traffic through a forward proxy. The proxy can block access to social media, log all traffic for compliance, and scan for malware.

**Anonymity and privacy** A user connects through a proxy to hide their real IP address from websites they visit.

**VPNs** A VPN is essentially a forward proxy with encryption. Your traffic goes to the VPN server (proxy) which forwards it to the destination. Websites see the VPN server's IP, not yours.

**Bypassing geographic restrictions** Connect through a proxy in another country to access content restricted to that region.

---

### Reverse Proxy — Hiding the Server

A **reverse proxy** sits between the **internet and your servers**. Clients send requests to the reverse proxy, which forwards them to the appropriate backend server. Clients see only the proxy — they have no idea how many servers sit behind it, what technology they run, or where they are.

```
Client → Internet → Reverse Proxy → Backend Servers
```

**Use cases:**

**Load Balancing** The reverse proxy distributes incoming requests across multiple backend servers. If one server is overwhelmed, requests go to others. If one server fails, it is removed from rotation automatically.

**SSL Termination** HTTPS connections are terminated at the reverse proxy. The proxy decrypts the traffic and forwards plain HTTP to the backend. Backend servers do not need to handle encryption — the proxy manages all certificates centrally.

**Caching** The reverse proxy can cache responses from backend servers. If 1,000 users request the same product page, the backend handles it once — the proxy serves the cached version to the other 999.

**Compression** The reverse proxy can compress responses before sending them to clients — reducing bandwidth usage transparently.

**Security** The reverse proxy is the only publicly visible component. Backend servers are on private networks — they cannot be directly attacked. The proxy can also block malicious IPs, enforce rate limits, and filter bad requests before they reach your application.

**Common reverse proxies:** Nginx, HAProxy, Apache, Caddy, AWS ALB (Application Load Balancer), Cloudflare.

---

### API Gateway — The Intelligent Reverse Proxy

An **API Gateway** is a specialised reverse proxy designed for microservice architectures. It does everything a reverse proxy does — and much more.

**What an API Gateway adds:**

- **Authentication and Authorisation** — verify JWT tokens, API keys, OAuth before requests reach your services
- **Rate Limiting** — enforce per-user or per-IP request limits
- **Request transformation** — modify request/response format, add headers, translate between API versions
- **Service routing** — route `/api/users` to the user service, `/api/orders` to the order service
- **Observability** — centralised logging, metrics, and tracing for all API traffic
- **Circuit breaking** — stop sending requests to a failing service

**Real examples:** AWS API Gateway, Kong, Nginx (with plugins), Traefik, Apigee.

> [!example] Production Architecture A real production system might look like: `User → Cloudflare (DDoS protection) → AWS ALB (SSL termination, load balancing) → Kong API Gateway (auth, rate limiting, routing) → Microservices` Each layer adds specific value. This is not overengineering — it is defence in depth.

---

## 📊 A10 — Network Fundamentals That Show Up in Interviews

These are the concepts that appear in System Design interviews without warning. You are expected to know them and reason about them fluently.

---

### Latency Numbers Every Engineer Should Know

Jeff Dean of Google published these numbers. They are approximate, they change slightly as hardware improves, but the **order of magnitude** relationships between them are what matter — and they have remained consistent for decades.

|Operation|Approximate Latency|
|---|---|
|L1 cache reference|0.5 ns|
|L2 cache reference|7 ns|
|Main memory (RAM) access|100 ns|
|SSD random read|150 µs|
|Read 1MB sequentially from SSD|1 ms|
|Round trip within same data center|0.5 ms|
|HDD seek time|10 ms|
|Round trip Mumbai to London|~120 ms|
|Read 1MB sequentially from HDD|20 ms|

**What these numbers tell a system designer:**

Memory is **200x faster** than SSD. SSD is **67x faster** than HDD for random access. A network round trip within a data center is 0.5ms — but a cross-continent round trip is 120ms.

If your service makes 10 cross-database calls per request, each adding 1ms, your response time floor is at least 10ms — before any processing. Design accordingly.

---

### Idempotency

An operation is **idempotent** if performing it multiple times produces the same result as performing it once.

**Why it matters:** Networks are unreliable. A client sends a request, the network drops the response. The client does not know if the request succeeded. So it retries. If the operation is not idempotent, retrying causes problems.

**Example — Payment processing:** User clicks "Pay ₹500." The request reaches the server. The payment is processed. The response times out before reaching the client. The client retries. Without idempotency — the user is charged twice.

**The solution — Idempotency keys:** The client generates a unique ID (UUID) for the request and sends it as an `Idempotency-Key` header. The server checks: _"Have I already processed a request with this key?"_ If yes — return the stored result. If no — process and store. Now retries are safe.

Stripe, Razorpay, and every serious payment API implement idempotency keys. You should too, for any non-idempotent operation that might be retried.

---

### Timeouts and Retries

**Timeouts** Every network call in a production system must have a timeout. Without a timeout, a slow or unresponsive upstream service can cause your thread/connection to wait forever — eventually exhausting all your resources and taking your own service down.

Set timeouts that reflect your SLA. If your API must respond in 500ms, your database calls should time out at 200ms, leaving time for processing and the response.

**Retries** When a network call fails (timeout, 503, 502), retrying automatically can recover from transient failures. But naive retries make problems worse.

**Exponential Backoff with Jitter:** Do not retry immediately and do not retry at fixed intervals. Wait a random amount of time that grows exponentially: 1s, 2s, 4s, 8s... The randomness (jitter) prevents all clients from retrying simultaneously and amplifying the load on an already struggling service.

---

### Circuit Breaker Pattern

**The problem:** Service A calls Service B. Service B starts responding slowly. Service A's threads pile up waiting. Service A runs out of threads. Service A crashes. Now a problem in Service B has taken down Service A too. This is **cascading failure** — one of the most dangerous failure modes in distributed systems.

**The circuit breaker** is the solution. Named after the electrical component that breaks a circuit when current is too high.

The circuit breaker tracks failure rate for calls to Service B. It has three states:

**Closed (normal operation)** — requests flow through. Failures are counted.

**Open (tripped)** — failure rate exceeded the threshold. ALL requests to Service B are immediately rejected without being made. Service A returns an error or a fallback response instantly. Service B has time to recover. No threads are wasted waiting.

**Half-Open (testing recovery)** — after a timeout, a small number of test requests are allowed through. If they succeed, the circuit closes. If they fail, it opens again.

> [!example] Real World Netflix's Hystrix library popularised the circuit breaker pattern. When their recommendation service was unavailable, instead of failing the entire page load, the circuit opened and the page showed generic popular titles as a fallback. Partial functionality beats complete failure.

---

### Long Tail Latency — p50, p95, p99

When measuring system performance, **averages are dangerous**.

Imagine a service where:

- 990 out of 1000 requests respond in 10ms
- 10 requests respond in 10,000ms (10 seconds)

The **average** is: `(990 × 10 + 10 × 10000) / 1000 = 109ms`

109ms sounds reasonable. But 1% of your users are waiting 10 full seconds. That is tens of thousands of users daily at scale.

**Percentiles (p) tell the real story:**

- **p50 (median)** — 50% of requests are faster than this. 10ms.
- **p95** — 95% of requests are faster than this. Typically your "mostly good" threshold.
- **p99** — 99% of requests are faster than this. Your "worst normal case."
- **p99.9** — 99.9% of requests are faster than this. Your "bad day" number.

**Why the tail is often caused:**

- Garbage collection pauses in Java/Go
- Database queries hitting cold cache
- Network retransmissions
- Lock contention

> [!tip] Interview Gold When an interviewer asks about your system's performance, never say "our average latency is 50ms." Say "our p99 latency is 120ms and our p99.9 is 450ms." That signals you understand distributed systems and real production behaviour.

---

> [!success] Part A — Complete You now understand the network layer that every system runs on. DNS, TCP, HTTP, TLS, WebSockets, APIs, Proxies — these are not abstract concepts. They are the wires and agreements that hold every system you will ever design together. In Part B, we go deeper — into the machines that run your code.

---

_Next →_ [[02-Foundations-of-System-Design/CS-Fundamentals-Part-B|Chapter 2 — Part B: Operating System Concepts]]

---

_— Sarvan Yaduvanshi · Competitive Programmer · LLM Foundation Model Engineer_