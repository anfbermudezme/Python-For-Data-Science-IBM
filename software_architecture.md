# Software Architecture: The Hard Parts — Summary

This README is a practical guide for **Software Architecture: The Hard Parts** (Neal Ford, Mark Richards, Pramod Sadalage, Zhamak Dehghani).

---

## Table of Contents
- [1. The core idea: architecture is trade-offs](#1-the-core-idea-architecture-is-trade-offs)
- [2. Key mental models](#2-key-mental-models)
- [3. Decomposition and granularity](#3-decomposition-and-granularity)
- [4. Data-first architecture](#4-data-first-architecture)
- [5. Distributed transactions & eventual consistency](#5-distributed-transactions--eventual-consistency)
- [6. Workflows: orchestration vs choreography](#6-workflows-orchestration-vs-choreography)
- [7. Saga pattern matrix (the 3D decision grid)](#7-saga-pattern-matrix-the-3d-decision-grid)
- [8. Contracts, versioning, and consumer-driven contracts](#8-contracts-versioning-and-consumer-driven-contracts)
- [9. Architecture governance that scales: ADRs + fitness functions](#9-architecture-governance-that-scales-adrs--fitness-functions)
- [10. Analytical data & data mesh](#10-analytical-data--data-mesh)
- [Glossary](#glossary)

---

## 1. The core idea: architecture is trade-offs
This book rejects “always do X” rules. Instead, it teaches you to:
1. **Find** what is entangled (what changes together),
2. **Analyze** coupling (static + dynamic),
3. **Assess** trade-offs (cost of change, scale, availability, team boundaries),
4. **Decide** and record it (ADRs),
5. **Enforce** it continuously (fitness functions).

Key message: modern architecture is often **data-driven**—data ownership and data movement decisions frequently dominate the final architecture.

---

## 2. Key mental models

### 2.1 Architecture Quantum
An **architecture quantum** is the *smallest* unit that can be:
- **Independently deployed**, and
- **Highly cohesive** (features inside belong together)

A system can look like microservices, but still behave like **one quantum** if shared dependencies force lockstep deployment (e.g., shared database, shared UI coupling, central mediator).

**Why it matters:**  
Quanta determine how “independent” your system really is. Many architectural failures are “fake microservices” caused by hidden coupling.

---

### 2.2 Static vs Dynamic Coupling
- **Static coupling**: deployment and infrastructure coupling  
  Examples: shared databases, shared runtime, shared orchestrator, shared schema, shared libraries that force coordinated releases.
- **Dynamic coupling**: runtime interaction coupling  
  Determined by three forces (next section).

---

### 2.3 The Three Forces (“3C’s”)
For any cross-service workflow, you must choose:
- **Communication**: synchronous vs asynchronous
- **Consistency**: atomic vs eventual
- **Coordination**: orchestration vs choreography

These three decisions strongly constrain each other.

---

## 3. Decomposition and granularity

### 3.1 Modularity vs Granularity
- **Modularity** = “how you split the system into parts”
- **Granularity** = “how big each part is”

You can have modularity without microservices (e.g., a modular monolith).  
Granularity is about practical boundary sizing.

---

### 3.2 Granularity: Integrators vs Disintegrators
A useful decision lens:

**Disintegrators** (push toward smaller services):
- Independent scaling needs
- Fault isolation
- Security boundaries
- High change/volatility in a subset of functionality
- Different release cadence
- Extensibility (plugins, optional features)

**Integrators** (push toward larger services):
- Strong data relationships
- Need for simple transactions
- High runtime chat/latency between split parts
- Shared domain logic is heavy or frequently changing together
- Complexity cost of distributed workflows is too high

**Rule of thumb:** split only when the *benefits* outweigh the added coupling + operational complexity.

---

## 4. Data-first architecture

### 4.1 Data is a primary driver of boundaries
Service boundaries often succeed or fail based on data:
- Who **owns** data (writes it)?
- Who needs to **read** it?
- How consistent must it be?
- What happens under failure?

### 4.2 Data ownership principle
**The service that writes the data should own it.**  
Other services should access it via:
- API calls,
- events + local read models,
- replication strategies,
not by direct database access.

### 4.3 Breaking up shared databases
Shared databases create major coupling:
- performance bottlenecks,
- scaling constraints,
- single points of failure,
- forced coordination across teams.

Typical refactoring techniques:
- **Table splitting** (move a subset to the owning domain)
- **Data domains** (reshape storage around bounded contexts)
- **Delegation** (temporary “owner service” writes on behalf of others during migration)
- **Service consolidation** (if separation is fighting reality)

---

## 5. Distributed transactions & eventual consistency

### 5.1 Why “atomic across services” is hard
Distributed systems can’t rely on “one transaction to rule them all” without sacrificing availability and operational simplicity.  
So cross-service business operations usually become **distributed transactions**.

### 5.2 Atomic vs Eventual (the difference)
- **Atomic**: all-or-nothing *now*. Either every step succeeds, or everything rolls back.
- **Eventual**: each step commits independently; the system becomes consistent **over time** via retries, compensations, and reconciliation.

### 5.3 Eventual consistency patterns
Common ways to achieve convergence:
1. **Background synchronization**  
   Periodic reconciliation jobs fix drift.
2. **Orchestrated request-based**  
   A coordinator drives the workflow and handles retries/compensation.
3. **Event-based propagation (pub/sub)**  
   Services react to events and update their own state; retries and idempotency are critical.

---

## 6. Workflows: orchestration vs choreography

### 6.1 Orchestration
A central workflow controller (“orchestrator”) tells participants what to do.

**Pros**
- Clear visibility into state
- Centralized error handling and compensation logic
- Easier to reason about complex workflows

**Cons**
- Risk of a “smart hub” becoming a bottleneck or mini-monolith
- More centralized ownership/team dependency

**Use when**
- Workflow is complex,
- Error paths matter,
- Strong observability and recoverability are needed.

---

### 6.2 Choreography
No central controller. Services publish events; other services react.

**Pros**
- High scalability and decoupling
- Participants evolve more independently

**Cons**
- Harder to reason about end-to-end flow
- Debugging and state tracking require extra tooling
- Error handling is distributed and often more complex

**Use when**
- Workflow is simple,
- High throughput matters,
- Errors are rare or can be handled locally.

---

## 7. Saga pattern matrix (the 3D decision grid)

A powerful framework: classify distributed workflows by the 3 forces:

- Communication: **Sync / Async**
- Consistency: **Atomic / Eventual**
- Coordination: **Orchestrated / Choreographed**

This produces **8 saga “families”**. You don’t memorize names to sound smart—you use the matrix to force explicit trade-offs.

### 7.1 The 8 combinations
| Communication | Consistency | Coordination | Practical interpretation |
|---|---|---|---|
| Sync | Atomic | Orchestrated | “Feels like a transaction,” but pushes complexity into coordinator |
| Sync | Atomic | Choreographed | Very brittle; atomicity + distributed control is painful |
| Sync | Eventual | Orchestrated | Coordinated steps, but accepts temporary inconsistency |
| Sync | Eventual | Choreographed | Sync calls but relaxed consistency; can still be tightly coupled |
| Async | Atomic | Orchestrated | Coordination via async, still trying to keep atomic behavior |
| Async | Atomic | Choreographed | Generally an anti-pattern: hard to ensure atomicity without control |
| Async | Eventual | Orchestrated | Good control with async steps; explicit compensation logic |
| Async | Eventual | Choreographed | Highly scalable; requires strong event discipline + idempotency |

**Key takeaway:**  
As you increase decoupling (async + choreography), you usually accept eventual consistency.

---

## 8. Contracts, versioning, and consumer-driven contracts

### 8.1 Strict vs loose contracts
- **Strict contracts** (e.g., strongly typed RPC): high safety & performance, but can increase coordination cost and break consumers more easily.
- **Loose contracts** (e.g., REST/JSON, schema-flexible): easier evolution, but require strong testing and governance to avoid drift.

### 8.2 Versioning
Versioning isn’t just a technical decision; it’s a coordination strategy.
Good versioning minimizes forced synchronized deployments.

### 8.3 Consumer-driven contracts (CDC)
Consumers publish tests or expectations; providers must pass them in CI/CD.

**Why it matters:**  
CDC turns “contract correctness” into an automated, enforceable constraint—great for microservices where many consumers exist.

---

## 9. Architecture governance that scales: ADRs + fitness functions

### 9.1 Architecture Decision Records (ADRs)
ADRs capture:
- Context
- Decision
- Alternatives considered
- Consequences (good + bad)
- Revisit triggers

They prevent “tribal knowledge architecture.”

### 9.2 Architecture fitness functions
Fitness functions are automated checks that verify architectural qualities continuously:
- dependency rules (no forbidden coupling),
- layering enforcement,
- latency budgets,
- database connection limits,
- API backward compatibility,
- event schema validation.

**Big idea:** governance should be **executable**, not just documentation.

---

## 10. Analytical data & data mesh

Operational systems optimize for transactions; analytical systems optimize for insight.  
The book introduces modern analytical architecture options and explains **data mesh** at a practical level.

### Data mesh (in one line)
A sociotechnical architecture where domains publish “data products” with clear ownership, contracts, and quality guarantees—like microservices, but for analytical data.

---

## Glossary
- **Architecture Quantum**: smallest independently deployable cohesive unit.
- **Static coupling**: deployment/infrastructure coupling.
- **Dynamic coupling**: runtime coupling (communication/coordination/consistency).
- **3C’s**: communication, consistency, coordination.
- **Atomic consistency (ACID)**: all-or-nothing now.
- **Eventual consistency (BASE)**: converges over time.
- **Saga**: distributed transaction pattern using coordination + compensation.
- **Orchestration**: central workflow controller.
- **Choreography**: event-based distributed control.
- **ADR**: record of architectural decisions.
- **Fitness function**: automated check enforcing architectural qualities.
