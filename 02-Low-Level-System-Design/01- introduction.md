# System Design — From Zero to Architect

_— Sarvan Yaduvanshi_

> Competitive Programmer · LLM Foundation Model Engineer · System Design Enthusiast

---

> [!quote] _"The quality of a system is determined not by its grand architecture, but by the precision of its smallest decisions."_

---

# Chapter 2 — Low Level System Design

---

## 🔩 What is Low Level System Design?

Low Level System Design is the process of **defining the internal structure of individual components** — how classes are written, how objects interact, how data flows within a single service, and which design patterns are applied to solve specific problems.

If High Level Design is the **city map**, then Low Level Design is the **engineering blueprint of a single building** — every floor, every room, every wall, every wire.

Think of it this way — imagine you are building a **Library Management System**. The High Level Design would say:

- We need a database
- We need a backend server
- We need an API layer

But Low Level Design answers:

- What classes will we create? — `Book`, `Member`, `Librarian`, `IssuedBook`
- How do they relate to each other?
- What methods will each class have?
- If two members request the same book at the same time — what happens?
- Which design pattern handles the notification when a book becomes available?

**LLD is where your object-oriented thinking gets tested.**

> [!example] Real Example — Parking Lot System **HLD says:** We need a database, a backend, and an entry/exit system. **LLD says:** Create a `ParkingLot` class with a `Vehicle` hierarchy — `Car`, `Bike`, `Truck`. Each `ParkingSpot` has a type and availability status. A `TicketSystem` generates `ParkingTicket` objects. A `PaymentService` handles billing using the **Strategy Pattern** so we can switch between cash, card, and UPI without changing the core logic. That level of detail — that is LLD.

---

## 🎯 Why Learn Low Level System Design?

Most developers can write code. But very few can write code that is **clean, scalable, maintainable, and extensible**. That gap is exactly what LLD fills.

### Because bad design is expensive

Imagine you join a company. In the first week you open the codebase and see:

- 2,000-line classes doing everything
- No separation of concerns
- A change in one place breaks five other things
- Adding a new payment method means touching 12 different files

This is what happens when a system is built **without Low Level Design**. The code works on day one. But in six months, the team is spending more time fixing bugs than building features.

**Good LLD prevents this from the start.**

### Real-world reasons LLD matters

**1. It is what you do every single day as a developer** HLD decisions are made once. LLD decisions are made hundreds of times — every class, every method, every module. Getting them right is what makes a codebase healthy for years.

**2. It is tested directly in interviews** Companies like Google, Microsoft, Amazon, Flipkart, and Uber have dedicated **Machine Coding rounds** where you are given 60–90 minutes to write production-quality code for a system. No LLD knowledge = no pass.

**3. It makes you a better HLD thinker** You cannot design a system at the macro level if you do not understand how its components work at the micro level. LLD and HLD are not opposites — they are the same skill at different zoom levels.

**4. It separates good engineers from great ones** Any engineer can add a feature. A great engineer adds a feature in a way that makes the _next_ feature easier to add. That is only possible with solid LLD thinking.

> [!warning] The Hard Reality In a Machine Coding interview, writing code that "works" is not enough. The interviewer is watching whether your classes are clean, your responsibilities are separated, and your design can handle change without breaking. LLD is the difference between passing and failing that round.

---

## 🥇 Why Learn LLD Before HLD?

This question comes up for every beginner. The answer is not opinion — it is logic.

|Reason|Explanation|
|---|---|
|**LLD is closer to code**|You already know how to write functions and classes. LLD builds directly on top of that. HLD requires understanding many systems you have never built yet.|
|**LLD gives HLD meaning**|When an HLD diagram shows a "Service" box — LLD is what lives inside that box. Without LLD, HLD is just shapes on a whiteboard.|
|**Interview sequence**|For junior and mid-level roles, Machine Coding rounds come before System Design rounds. LLD gets you through the door.|
|**Concepts carry over**|SOLID principles, design patterns, and clean architecture — all LLD concepts — show up constantly in HLD discussions.|
|**Confidence building**|Mastering LLD gives you the vocabulary, thinking patterns, and confidence to tackle the abstract challenges of HLD.|

> [!success] The Right Order Learn LLD → understand how components work deeply → then learn HLD → now you can connect the dots at scale. Skipping LLD to rush into HLD is like learning multiplication before addition. It will catch up with you.

---

## 📋 Chapter 2 — Table of Contents

Everything in this chapter will be covered fully. No topic is skipped. No shortcut is taken.

---

### Part A — Object Oriented Programming

> The foundation everything else is built on.

- **OOP Core Concepts** — Class, Object, Method, Attribute
- **The Four Pillars of OOP**
    - Encapsulation — hiding internal state
    - Abstraction — exposing only what is necessary
    - Inheritance — building on existing behaviour
    - Polymorphism — one interface, many implementations
- **When OOP Goes Wrong** — common beginner mistakes that break design

---

### Part B — SOLID Principles

> Five rules that separate amateur code from professional code.

- **S — Single Responsibility Principle** — one class, one job
- **O — Open / Closed Principle** — open for extension, closed for modification
- **L — Liskov Substitution Principle** — subtypes must be substitutable for their base types
- **I — Interface Segregation Principle** — do not force classes to implement what they do not use
- **D — Dependency Inversion Principle** — depend on abstractions, not concretions
- **SOLID Violations** — real examples of what broken code looks like

---

### Part C — Design Patterns

> Battle-tested solutions to common design problems.

**Creational Patterns** — how objects are created

- Singleton Pattern
- Factory Pattern
- Abstract Factory Pattern
- Builder Pattern
- Prototype Pattern

**Structural Patterns** — how objects are composed

- Adapter Pattern
- Decorator Pattern
- Facade Pattern
- Proxy Pattern
- Composite Pattern
- Bridge Pattern
- Flyweight Pattern

**Behavioural Patterns** — how objects communicate

- Strategy Pattern
- Observer Pattern
- Command Pattern
- Iterator Pattern
- State Pattern
- Template Method Pattern
- Chain of Responsibility Pattern
- Mediator Pattern

---

### Part D — UML Diagrams

> The language designers use to communicate structure without writing code.

- **Class Diagrams** — relationships between classes
- **Sequence Diagrams** — how objects interact over time
- **Use Case Diagrams** — what the system does from a user's perspective
- **Activity Diagrams** — flow of logic and decisions
- **How to Read and Draw UML** — practical, interview-ready approach

---

### Part E — LLD Case Studies

> Where all the theory becomes real. Each case study is designed end-to-end.

- **Parking Lot System** — the classic starter case study
- **Library Management System** — inheritance, relationships, search
- **Chess Game** — movement rules, turn management, win conditions
- **Snake and Ladder** — game engine design, dice, board state
- **Elevator System** — scheduling algorithms, multi-lift coordination
- **Hotel Booking System** — availability, reservations, payments
- **BookMyShow / Movie Ticket Booking** — seats, shows, concurrency
- **ATM System** — state machine, transactions, security
- **Food Delivery App — Zomato / Swiggy** — order lifecycle, real-time tracking
- **Ride Sharing — Uber / Ola** — driver matching, pricing, trip management

---

> [!success] What You Will Be Able to Do After This Chapter Design any system at the component level from scratch. Walk into a Machine Coding round with confidence. Write code that your team will thank you for — not curse you for.

---

_Next →_ [[02-Low-Level-System-Design/OOP-Core-Concepts|Chapter 2.1 — OOP Core Concepts]]

---

_— Sarvan Yaduvanshi · Competitive Programmer · LLM Foundation Model Engineer_