# Microservices: Up and Running — Team Summary & Practical Playbook

This README is a **resume** of the book *Microservices: Up and Running* (Ronnie Mitra, Irakli Nadareishvili).

---

## Why microservices (the “real” reason)

Microservices are not a fashion statement. They’re a strategy to **reduce coordination costs** in complex systems:
- Less waiting on other teams
- Fewer giant synchronized releases
- Smaller “blast radius” when changes go wrong
- Faster iteration with clearer ownership

Trade-off: microservices shift complexity **into architecture + operations** (deployment, monitoring, distributed data, etc.). You don’t delete complexity; you **move it**.

---

## The book’s end-to-end model (what it builds)

The “Up & Running” approach is an opinionated, practical path:

1. **Operating model** (teams, ownership, collaboration)
2. **Service design** (how to design APIs/services consistently)
3. **Right-sizing boundaries** (where to split services)
4. **Data** (independent deployability without shared databases)
5. **Infrastructure pipeline** (IaC + CI/CD)
6. **Runtime platform** (Kubernetes + GitOps)
7. **Developer workspace + implementation**
8. **Release & change management**
9. **Measurement** (to prove this is working)

Example domain used through the book: a small airline system with two services:
- `ms-flights` (flight information)
- `ms-reservations` (seat reservations)

---

## Key concepts your team should share

### ADR — Architectural Decision Record
A lightweight way to record important decisions so future-you doesn’t suffer.

Minimum structure:
- **Context:** what problem / constraints existed?
- **Alternatives:** what other options were considered?
- **Decision:** what did we choose?
- **Consequences:** trade-offs and impacts (what gets harder/easier)

Use ADRs whenever you make a “fork-in-the-road” choice:
- boundaries, tech choices, data strategy, deployment style, API versioning, etc.

---

### Team Topologies (operating model)
Microservices succeed when team structure matches system structure.

Common team types:
- **Stream-aligned teams:** own a product slice end-to-end (build + run)
- **Platform team:** provides “paved road” infrastructure (CI/CD, K8s, observability, templates)
- **Enabling team:** helps other teams adopt practices (temporary coaching)
- **Complicated-subsystem team:** owns specialized areas (rare; use when needed)

Key idea: design interactions intentionally (collaboration vs X-as-a-service).

---

### SEED(S) — Seven Essential Evolutions of Design for Services
A repeatable, customer-centric process to design **service interfaces** (APIs) before code.

1. Identify **actors** (who uses the service)
2. Identify **jobs-to-be-done** (JTBD) using job stories
3. Map interactions using **sequence diagrams**
4. Derive **actions (commands)** and **queries**
5. Write specs in an open standard (**OpenAPI / GraphQL**)
6. Get feedback (design review with consumers)
7. Implement the microservice using the agreed contract

Output: APIs that are intentional, reviewable, and easier to evolve.

---

### DDD — Domain-Driven Design (boundaries & language)
DDD is used to find boundaries that match the business.

Core ideas:
- **Bounded Context:** each domain area has its own model and meaning of terms
- **Ubiquitous Language:** shared vocabulary inside a context (engineers + domain experts)
- **Aggregate:** a consistency boundary; expose an “aggregate root” to outside callers
- **Context Mapping:** describe how contexts interact (Upstream/Downstream, ACL, Open Host Service)

DDD helps you avoid the trap of one universal “canonical model” that everyone fights over.

---

### Data independence (no shared databases)
Microservice independence usually requires **data ownership**:
- Each service owns its data and schema
- Avoid “shared DB” because it creates hidden coupling and coordinated deployments

Patterns to keep autonomy without chaos:
- **Data delegation:** a service provides data through an API instead of DB sharing
- **Data duplication:** copy what you need (carefully) to avoid runtime coupling
- **Sagas / compensations:** handle multi-step workflows across services without distributed ACID transactions

---

### Event Sourcing + CQRS (advanced tools, not default)
When you need auditability, loose coupling, or powerful projections:

- **Event Sourcing:** store immutable events as the source of truth (“facts happened”)
- **Event Store:** append-only log of events
- **Projections / read models:** derived views built from events
- **CQRS:** separate **write model** (commands/events) from **read model** (queries/indexes)

Benefits:
- Strong history/audit trail
- Rebuild/repair read views from events
- Scale reads and writes independently

Cost:
- More moving parts
- Harder debugging & operational complexity  
Use only when the value is real.

---

## Platform & delivery approach

### IaC + CI/CD + Immutable infrastructure
- Define infrastructure as code (e.g., Terraform)
- Use pipelines (e.g., GitHub Actions) to apply infrastructure changes
- Prefer immutable infra practices (recreate rather than patch in place)
- Promote changes across environments (sandbox → staging → prod)

### Kubernetes + GitOps
- Run services on Kubernetes
- Use GitOps (e.g., Argo CD): **Git is the source of truth**
- Cluster state is continuously reconciled to what’s declared in Git

---

## Implementation & release mechanics

Typical flow:
1. Implement services (following API specs from SEED(S))
2. Build container images
3. Push to a registry
4. Deploy with Helm charts / manifests
5. Argo CD syncs from Git to the cluster

Change & rollout patterns:
- **Rolling updates** (simple)
- **Blue/Green** (two environments; fast rollback)
- **Canary** (small traffic first; then expand)

Schema/API evolution techniques:
- Backward-compatible changes
- Expand/contract patterns for data migrations
- Contract testing when teams deploy independently

---

## Measurement: “Is this transformation actually working?”
Track operational + delivery metrics (DORA-style and coordination-cost signals):

- **Lead time** (idea → production)
- **Deployment frequency**
- **Mean time to restore** (MTTR)
- **Change failure rate**
- **Coordination indicators:** dependencies per release, waiting time on other teams, stoppages caused by shared data changes

Microservices only win if these numbers improve over time.

---

## Full book map diagram

```mermaid
flowchart TD

%% ================== CH1: FOUNDATIONS ==================
subgraph C1["Ch1 – Foundations: Microservices & Decisions"]
  C1A["Goal: faster delivery *with* safety<br/>(microservices as small, independent components)"]
  C1B["Core problem: coordination costs<br/>between teams & services"]
  C1C["Fictional domain: Airline reservations<br/>- Flight info service<br/>- Seat reservation service"]
  C1D["“Up & Running” model:<br/>Team design • Service design • Data • Platform • Dev • Release • Change & measurement"]
  C1E["Architectural Decision Records (ADRs)<br/>– lightweight text files to log key decisions"]
end

%% ================== CH2: OPERATING MODEL ==================
subgraph C2["Ch2 – Operating Model: Teams & Topologies"]
  C2A["Team Topologies:<br/>- Stream-aligned teams<br/>- Platform teams<br/>- Enabling teams<br/>- Complicated-subsystem teams"]
  C2B["System Design Team<br/>(sets overall architecture & principles)"]
  C2C["Stream-Aligned ‘Microservice’ Teams<br/>(build & own ms-flights, ms-reservations, etc.)"]
  C2D["Cloud Platform Team<br/>(provides network, k8s, deployment as a service)"]
  C2E["API / BFF Team<br/>(consumer-facing APIs over microservices)"]
  C2F["Release / Operations Team<br/>(observability, incidents, release discipline)"]
end

C1D --> C2A
C1E --> C2B
C1B --> C2B

%% ================== CH3–4: SERVICE DESIGN & BOUNDARIES ==================
subgraph C3["Ch3–4 – Service Design: SEED(S), DDD & Boundaries"]
  C3A["SEED(S) method:<br/>Seven Essential Evolutions of Design for Services"]
  C3B["1. Identify actors<br/>(personas: customer, app, BFF API, microservices, etc.)"]
  C3C["2. Identify Jobs-To-Be-Done (JTBDs)<br/>using Job Stories"]
  C3D["3. Map interactions with sequence diagrams"]
  C3E["4. Derive actions (commands) & queries<br/>from JTBDs & flows"]
  C3F["5. Specify APIs with open standards<br/>(OpenAPI / GraphQL schemas)"]
  C3G["6. Get feedback on the API spec<br/>(consumer-first design loop)"]
  C3H["7. Implement microservices<br/>(based on the agreed contracts)"]

  C3I["Domain-Driven Design (DDD):<br/>- Bounded contexts<br/>- Ubiquitous language<br/>- Aggregates"]
  C3J["Event Storming workshops<br/>(events → domains → candidate services)"]
  C3K["Universal sizing formula<br/>(guide for service granularity)"]
end

C2C --> C3A
C2A --> C3I
C3A --> C3B --> C3C --> C3D --> C3E --> C3F --> C3G --> C3H
C3I --> C3J --> C3K

%% ================== CH5: DATA ==================
subgraph C4["Ch5 – Data: Independence, Delegates, Event Sourcing, CQRS"]
  C4A["Principle: each microservice *owns & embeds* its data<br/>(no shared DBs, independent deployability)"]
  C4B["Data delegate pattern & duplication<br/>(to avoid cross-service joins via shared DBs)"]
  C4C["Distributed transactions & Sagas<br/>(compensating actions across services)"]
  C4D["Event Sourcing:<br/>store events (“facts”) instead of mutable state"]
  C4E["Event Store<br/>(append-only log of domain events)"]
  C4F["Snapshots + Projections<br/>(rebuild state & precompute read models)"]
  C4G["CQRS – Command Query Responsibility Segregation:<br/>separate write model (event store) from read models (indices)"]
end

C3I --> C4A
C3H --> C4A
C4A --> C4B --> C4C
C4A --> C4D --> C4E --> C4F --> C4G

%% ================== CH6–7: PLATFORM & INFRA ==================
subgraph C5["Ch6–7 – Platform: IaC, AWS, Kubernetes, GitOps"]
  C5A["DevOps principles:<br/>CI/CD, small batch sizes, fast feedback"]
  C5B["Immutable Infrastructure & Infrastructure as Code (IaC)<br/>(Terraform + GitHub)"]
  C5C["AWS setup:<br/>- Org / Ops account<br/>- VPC, subnets, gateways<br/>- S3 backend for Terraform state"]
  C5D["Infrastructure pipeline (GitHub Actions):<br/>apply Terraform on every change<br/>→ sandbox/staging/prod environments"]
  C5E["Microservices Infrastructure:<br/>- Network module<br/>- Kubernetes (EKS) module<br/>- GitOps deployment server"]
  C5F["Argo CD as GitOps engine<br/>(watches git → syncs k8s resources)"]
end

C2D --> C5B
C4A --> C5C
C5B --> C5C --> C5D --> C5E --> C5F

%% ================== CH8–9: DEV EXPERIENCE & CODING ==================
subgraph C6["Ch8–9 – Developer Workspace & Microservice Code"]
  C6A["Developer workspace goals:<br/>fast onboarding, reproducible envs, good DX"]
  C6B["Workspace setup:<br/>Multipass / containers • Docker • local k8s (optional)"]
  C6C["Workspace guidelines:<br/>10 principles for a pleasant dev experience"]
  C6D["Microservices implemented:<br/>- ms-flights<br/>- ms-reservations"]
  C6E["API design first:<br/>OpenAPI specs based on SEED(S) outputs"]
  C6F["Data implementations:<br/>- Redis for reservations<br/>- MySQL for flights"]
  C6G["Service health checks<br/>(+ basic observability hooks)"]
  C6H["Umbrella project to run multiple microservices<br/>in a single dev workspace"]
end

C5D --> C6A
C6A --> C6B --> C6C
C3F --> C6E --> C6D
C4D --> C6F
C4G --> C6F
C6D --> C6G --> C6H

%% ================== CH10: RELEASING ==================
subgraph C7["Ch10 – Releasing Microservices"]
  C7A["Staging infrastructure:<br/>Ingress, DB module, k8s cluster"]
  C7B["Container images:<br/>Dockerfiles + Docker Hub registry"]
  C7C["Helm charts for each microservice<br/>(replicas, services, config, secrets)"]
  C7D["Release pipeline:<br/>GitHub Actions → build & push image →<br/>update git manifests → Argo CD syncs cluster"]
end

C5E --> C7A
C6D --> C7B --> C7C
C5F --> C7D
C7A --> C7D
C7C --> C7D

%% ================== CH11–12: CHANGE & TRANSFORMATION ==================
subgraph C8["Ch11–12 – Managing Change & Measuring Transformation"]
  C8A["Types of change:<br/>- Infrastructure<br/>- Microservices<br/>- Data & schemas"]
  C8B["Deployment patterns:<br/>blue/green • canary • rolling updates"]
  C8C["Be data‑oriented about change:<br/>use metrics & logs to drive decisions"]
  C8D["Microservices Quadrant:<br/>complex implementation / simple architecture<br/>vs monoliths (complicated implementation / easy-ish design)"]
  C8E["Measuring transformation:<br/>lead time, deployment frequency,<br/>MTTR, error rates, coordination cost"]
end

C7D --> C8A --> C8B --> C8C
C1B --> C8D --> C8E
```

---

## Glossary
- **ADR:** Architectural Decision Record
- **DDD:** Domain-Driven Design
- **SEED(S):** Seven Essential Evolutions of Design for Services
- **CQRS:** Command Query Responsibility Segregation
- **IaC:** Infrastructure as Code
- **GitOps:** Git as source of truth for deployments
- **Saga:** distributed workflow with compensating actions
