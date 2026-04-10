# System Design — From Zero to Architect

_— Sarvan Yaduvanshi_

> Competitive Programmer · LLM Foundation Model Engineer · System Design Enthusiast

---

> [!quote] _"Every great system was once just a thought in someone's head — the difference is how it was designed."_

---

# Chapter 1 — Introduction

---

## 📖 What is System Design?

System Design is the process of **defining the architecture, components, modules, interfaces, and data flow** of a system to satisfy a given set of requirements.

Think of it like this — before a city is built, engineers don't just randomly place roads and buildings. They **plan** it. They decide:

- Where will the roads go?
- How many lanes does a highway need?
- Where will the power stations be?
- What happens when one bridge breaks — does the whole city stop?

System Design is exactly that — but for **software**.

When you design a system, you're answering questions like:

- How does data move from the user to the database?
- What happens when 10 million people hit the app at the same time?
- Where do we store files — locally or in the cloud?
- How do we make sure the system doesn't go down?

> [!tip] Simple Way to Think About It System Design = **Planning how to build software** before actually building it. Like an architect drawing blueprints before construction begins.

---

## 🌍 Why System Design Matters

You might wonder — _"I can already code. Why do I need this?"_

Here's the truth: **coding is only 20% of the job at scale.**

A system that works for 100 users will **completely fall apart** at 10 million users — not because the code was wrong, but because nobody designed it to handle that load.

### Real reasons why System Design matters

**1. It saves money** A poorly designed system costs 10x more to fix later than to design correctly upfront.

**2. It is the difference between a Junior and a Senior engineer** Juniors write features. Seniors design systems. Companies pay $300K+ for engineers who can think at a system level.

**3. It is mandatory in tech interviews** Every top company — Google, Meta, Amazon, Netflix — has a dedicated System Design interview round. You cannot skip this.

**4. Real products need it** WhatsApp handles 100 billion messages per day. YouTube serves 500 hours of video per minute. None of this happens by accident — it is designed.

> [!warning] Hard Truth You can be an excellent coder and still fail a senior engineering interview because you do not understand system design. This book exists to change that.

---

## 🏗️ Real-World Applications

Let us ground this with examples you use every day:

|App|System Design Problem Solved|
|---|---|
|**WhatsApp**|Deliver messages instantly to billions of users across the world|
|**YouTube**|Store, compress, and stream petabytes of video efficiently|
|**Uber**|Match millions of riders and drivers in real-time across cities|
|**Instagram**|Show a personalized feed from 1 billion+ posts|
|**Google Search**|Search the entire internet and respond in under 0.5 seconds|
|**Spotify**|Stream music without buffering to 400 million users simultaneously|
|**Amazon**|Process millions of orders, manage inventory, and handle payments globally|

Every single one of these required someone to sit down and **design the system** before a single line of production code was written.

> [!example] Think About This When you send a WhatsApp message, it does not go directly to your friend. It travels through servers, gets queued, routed, encrypted, and delivered — all in under a second. That is system design in action.

---

## 🗂️ Types of System Design

System Design is broadly divided into **two types**:

### 1. High-Level Design — HLD

This is the **big picture** view. You are not writing any code here — you are drawing diagrams and making architectural decisions.

HLD answers questions like:

- What services will our system have?
- How will they communicate with each other?
- Which database should we use?
- How will we handle millions of requests at once?

_Think of HLD as the map of the city._

### 2. Low-Level Design — LLD

This is the **detailed** view. You are now zooming in to look at individual components — classes, methods, design patterns, object relationships.

LLD answers questions like:

- How will we structure our classes?
- Which design pattern should we use here?
- How will this specific module work internally?
- What does the database schema look like?

_Think of LLD as the blueprint for one specific building in the city._

> [!info] Quick Summary **HLD** = Architecture level — the forest **LLD** = Code level — the trees Both are essential. Both will be covered in this book — completely.

---

## ⚔️ HLD vs LLD — The Full Breakdown

This is one of the most common points of confusion for beginners. Let us settle it once and for all.

||**HLD**|**LLD**|
|---|---|---|
|**Focus**|Architecture and System|Classes and Code Structure|
|**Who does it?**|Architects, Senior Engineers|Developers, Engineers|
|**Tools used**|Diagrams, flowcharts|UML, code|
|**Question answered**|_What_ the system does|_How_ the system does it|
|**Examples**|Load balancer, Database choice, Caching|Class design, Design patterns, DB schema|
|**Interview round**|System Design Interview|Machine Coding / LLD Interview|

### Which do you learn first — HLD or LLD?

> [!question] The Classic Beginner Dilemma _"Should I learn LLD before HLD, or HLD before LLD?"_

**The honest answer: LLD first.**

Here is why:

- LLD is **closer to code** — you already know how to code, so it is a smoother transition
- Understanding _how_ individual components work makes HLD diagrams actually make sense
- In interviews, LLD rounds often come before System Design rounds for junior and mid-level roles
- Concepts like **SOLID principles, Design Patterns, OOP** — all LLD — show up inside HLD discussions too

That said, this book covers **both, completely**. You will not miss a single concept. The sequence in this book is designed so that each chapter builds naturally on the last.

> [!success] This Book's Approach Foundations → Low-Level Design → High-Level Design → Case Studies → Interview Prep. By the end, both will feel completely natural.

---

## 📚 How to Learn System Design

System Design is **not memorized — it is understood.**

Most people make the mistake of memorizing system designs of famous companies. That is useful later, but it will not help you design a new system from scratch in an interview.

### The right way to learn

**Step 1 — Build your vocabulary first** Know what a Load Balancer, Cache, CDN, Queue, and Database actually are and do.

**Step 2 — Understand the problems before the solutions** _Why_ do we need caching? _Why_ do we need sharding? Do not memorize — understand the problem being solved.

**Step 3 — Learn component by component** Go deep on each building block. How does a CDN work internally? What are the tradeoffs of SQL vs NoSQL?

**Step 4 — Practice designing systems** Start small. Design a URL shortener. Then a chat app. Then a video streaming service. Each one teaches you something new.

**Step 5 — Study real-world architectures** Netflix, Uber, Instagram have all published blogs about how their systems work. Reading these is like getting a masterclass for free.

**Step 6 — Practice explaining under pressure** Thinking about system design is one skill. Explaining it clearly in 45 minutes during an interview is a completely different skill. Practice both.

> [!tip] The Golden Rule Every time you use an app, ask yourself — _"How is this actually working under the hood?"_ That curiosity is what separates great engineers from average ones.

---

## 🗺️ System Design Roadmap

Here is your complete journey through this book. Every topic is a chapter. Nothing is skipped.

```
📦  FOUNDATIONS
    ├── What is System Design
    ├── HLD vs LLD
    └── How to Approach a Design Problem

🔩  LOW-LEVEL DESIGN
    ├── OOP Principles
    ├── SOLID Principles
    ├── Design Patterns — Creational, Structural, Behavioural
    ├── UML Diagrams
    └── LLD Case Studies — Parking Lot, Chess, BookMyShow...

🏛️  HIGH-LEVEL DESIGN — BUILDING BLOCKS
    ├── Scalability and Load Balancing
    ├── Caching — Redis, CDN
    ├── Databases — SQL vs NoSQL, Sharding, Replication
    ├── Message Queues and Event-Driven Architecture
    ├── API Design — REST, GraphQL, gRPC
    ├── Microservices vs Monolith
    ├── CAP Theorem
    ├── Rate Limiting
    ├── Distributed Systems
    └── Security Basics

🏗️  HIGH-LEVEL DESIGN — CASE STUDIES
    ├── Design a URL Shortener — like Bit.ly
    ├── Design a Chat App — like WhatsApp
    ├── Design a Video Platform — like YouTube
    ├── Design a Ride-Sharing App — like Uber
    ├── Design a Social Feed — like Instagram / Twitter
    ├── Design a Search Engine — like Google
    ├── Design a Payment System
    └── Design a Notification System

🎯  INTERVIEW PREP
    ├── How to Structure a System Design Answer
    ├── Common Mistakes to Avoid
    └── 30 Most Asked System Design Questions
```

> [!success] You Are Here Chapter 1 is complete. The foundation is set. Now we go deeper.

---

_Next →_ [[02-Low-Level-System-Design/LLD-Introduction|Chapter 2 — Introduction to Low-Level Design]]

---

_— Sarvan Yaduvanshi · Competitive Programmer · LLM Foundation Model Engineer_