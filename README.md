# Software Architecture: A Complete Tutorial

> A comprehensive, modularized reference covering every layer of software architecture — from first principles to distributed systems at scale.

---

## How to Use This Tutorial

Each topic is self-contained. Every file follows the same structure:

1. **The Problem** — why this exists and what breaks without it
2. **The Solution** — the concept explained in plain language
3. **How It Works** — deep mechanics, rules, sub-concepts
4. **Code Example** — runnable snippets with before/after comparisons
5. **Diagram** — Mermaid diagrams (class, sequence, flowchart, state)
6. **When to Use / When NOT to Use** — trade-offs and anti-patterns
7. **Key Takeaways** — 3–5 bullet summary

**Recommended reading order:** Follow the sections top-to-bottom if you are new. Jump directly to any section if you have a specific need.

---

## Table of Contents

### [00 — Fundamentals](./00-fundamentals/README.md)
> The universal rules every software system should obey, regardless of language or scale.

| # | Topic | What You'll Learn |
|---|-------|-------------------|
| 1 | [SOLID Principles](./00-fundamentals/01-solid-principles.md) | SRP, OCP, LSP, ISP, DIP |
| 2 | [DRY, KISS, YAGNI](./00-fundamentals/02-dry-kiss-yagni.md) | Avoiding duplication, over-engineering, and premature features |
| 3 | [Separation of Concerns](./00-fundamentals/03-separation-of-concerns.md) | Cohesion, coupling, the God-class anti-pattern |
| 4 | [Clean Code](./00-fundamentals/04-clean-code.md) | Naming, functions, comments, formatting |

---

### [01 — Design Patterns](./01-design-patterns/README.md)
> Proven, reusable solutions to recurring design problems.

#### Creational Patterns
| # | Pattern | Intent |
|---|---------|--------|
| 1 | [Singleton](./01-design-patterns/creational/01-singleton.md) | One instance, global access point |
| 2 | [Factory Method](./01-design-patterns/creational/02-factory-method.md) | Delegate object creation to subclasses |
| 3 | [Abstract Factory](./01-design-patterns/creational/03-abstract-factory.md) | Create families of related objects |
| 4 | [Builder](./01-design-patterns/creational/04-builder.md) | Construct complex objects step by step |
| 5 | [Prototype](./01-design-patterns/creational/05-prototype.md) | Clone existing objects instead of constructing new ones |

#### Structural Patterns
| # | Pattern | Intent |
|---|---------|--------|
| 1 | [Adapter](./01-design-patterns/structural/01-adapter.md) | Make incompatible interfaces work together |
| 2 | [Bridge](./01-design-patterns/structural/02-bridge.md) | Separate abstraction from implementation |
| 3 | [Composite](./01-design-patterns/structural/03-composite.md) | Treat individual objects and compositions uniformly |
| 4 | [Decorator](./01-design-patterns/structural/04-decorator.md) | Add behavior without subclassing |
| 5 | [Facade](./01-design-patterns/structural/05-facade.md) | Simplify a complex subsystem behind a single interface |
| 6 | [Flyweight](./01-design-patterns/structural/06-flyweight.md) | Share fine-grained objects to reduce memory |
| 7 | [Proxy](./01-design-patterns/structural/07-proxy.md) | Control access to another object |

#### Behavioral Patterns
| # | Pattern | Intent |
|---|---------|--------|
| 1 | [Chain of Responsibility](./01-design-patterns/behavioral/01-chain-of-responsibility.md) | Pass requests along a chain of handlers |
| 2 | [Command](./01-design-patterns/behavioral/02-command.md) | Encapsulate requests as objects |
| 3 | [Iterator](./01-design-patterns/behavioral/03-iterator.md) | Traverse a collection without exposing internals |
| 4 | [Mediator](./01-design-patterns/behavioral/04-mediator.md) | Centralize complex communications |
| 5 | [Memento](./01-design-patterns/behavioral/05-memento.md) | Capture and restore object state |
| 6 | [Observer](./01-design-patterns/behavioral/06-observer.md) | Notify dependents of state changes |
| 7 | [State](./01-design-patterns/behavioral/07-state.md) | Alter behavior when internal state changes |
| 8 | [Strategy](./01-design-patterns/behavioral/08-strategy.md) | Define a family of interchangeable algorithms |
| 9 | [Template Method](./01-design-patterns/behavioral/09-template-method.md) | Define skeleton of algorithm, defer steps to subclasses |
| 10 | [Visitor](./01-design-patterns/behavioral/10-visitor.md) | Add operations to objects without modifying them |

---

### [02 — Architecture Patterns](./02-architecture-patterns/README.md)
> High-level blueprints for structuring entire systems.

| # | Pattern | Intent |
|---|---------|--------|
| 1 | [Layered Architecture](./02-architecture-patterns/01-layered-architecture.md) | Organize code into horizontal layers |
| 2 | [MVC / MVP / MVVM](./02-architecture-patterns/02-mvc-mvp-mvvm.md) | Separate UI, logic, and data |
| 3 | [Clean Architecture](./02-architecture-patterns/03-clean-architecture.md) | Dependency rule + concentric layers |
| 4 | [Hexagonal Architecture](./02-architecture-patterns/04-hexagonal-architecture.md) | Ports & Adapters for maximum testability |
| 5 | [Event-Driven Architecture](./02-architecture-patterns/05-event-driven-architecture.md) | Communicate via events, not direct calls |
| 6 | [Microservices](./02-architecture-patterns/06-microservices.md) | Decompose a system into small, independent services |
| 7 | [Serverless](./02-architecture-patterns/07-serverless.md) | Functions as a unit of deployment |
| 8 | [CQRS](./02-architecture-patterns/08-cqrs.md) | Separate read and write models |
| 9 | [Event Sourcing](./02-architecture-patterns/09-event-sourcing.md) | Store state as a sequence of events |

---

### [03 — System Design](./03-system-design/README.md)
> How to design systems that are fast, reliable, and scalable.

| # | Topic | What You'll Learn |
|---|-------|-------------------|
| 1 | [Scalability](./03-system-design/01-scalability.md) | Vertical vs horizontal scaling, stateless design, sharding |
| 2 | [Load Balancing](./03-system-design/02-load-balancing.md) | Algorithms, health checks, sticky sessions |
| 3 | [Caching](./03-system-design/03-caching.md) | Strategies, eviction policies, Redis patterns |
| 4 | [Database Design](./03-system-design/04-database-design.md) | Normalization, indexing, N+1, NoSQL trade-offs |
| 5 | [API Design](./03-system-design/05-api-design.md) | REST, GraphQL, gRPC, versioning, rate limiting |
| 6 | [Message Queues](./03-system-design/06-message-queues.md) | Queue vs topic, delivery guarantees, backpressure |

---

### [04 — Distributed Systems](./04-distributed-systems/README.md)
> The hard problems that appear when services run on multiple machines.

| # | Topic | What You'll Learn |
|---|-------|-------------------|
| 1 | [CAP Theorem](./04-distributed-systems/01-cap-theorem.md) | Consistency, availability, partition tolerance |
| 2 | [Consistency Models](./04-distributed-systems/02-consistency-models.md) | Strong → eventual; linearizability |
| 3 | [Consensus Algorithms](./04-distributed-systems/03-consensus-algorithms.md) | Paxos intuition, Raft leader election |
| 4 | [Distributed Transactions](./04-distributed-systems/04-distributed-transactions.md) | 2PC, Saga pattern, compensating transactions |
| 5 | [Service Mesh](./04-distributed-systems/05-service-mesh.md) | Sidecar proxy, mTLS, traffic policies |
| 6 | [Circuit Breaker & Resilience](./04-distributed-systems/06-circuit-breaker-resilience.md) | Circuit states, bulkhead, retry with jitter |

---

### [05 — Testing](./05-testing/README.md)
> Strategies and patterns for building confidence in your code.

| # | Topic | What You'll Learn |
|---|-------|-------------------|
| 1 | [Testing Fundamentals](./05-testing/01-testing-fundamentals.md) | Testing pyramid, test doubles |
| 2 | [Unit Testing](./05-testing/02-unit-testing.md) | AAA pattern, what NOT to test |
| 3 | [Integration Testing](./05-testing/03-integration-testing.md) | DB tests, contract testing, testcontainers |
| 4 | [End-to-End Testing](./05-testing/04-e2e-testing.md) | Playwright, flakiness, page object model |
| 5 | [TDD](./05-testing/05-tdd.md) | Red-Green-Refactor, full worked example |
| 6 | [BDD](./05-testing/06-bdd.md) | Gherkin, Given-When-Then, Cucumber |

---

### [06 — Security Architecture](./06-security-architecture/README.md)
> Patterns for building systems that are secure by design.

| # | Topic | What You'll Learn |
|---|-------|-------------------|
| 1 | [Auth Patterns](./06-security-architecture/01-auth-patterns.md) | Session, JWT, OAuth2, OIDC |
| 2 | [Zero Trust](./06-security-architecture/02-zero-trust.md) | Never trust / always verify, microsegmentation |
| 3 | [OWASP Top 10](./06-security-architecture/03-owasp-top10.md) | Top 10 vulnerabilities + mitigations |

---

### [07 — DevOps & Infrastructure](./07-devops-infrastructure/README.md)
> Automating delivery and managing infrastructure reliably.

| # | Topic | What You'll Learn |
|---|-------|-------------------|
| 1 | [CI/CD](./07-devops-infrastructure/01-cicd.md) | Pipeline stages, trunk-based dev, deploy strategies |
| 2 | [Infrastructure as Code](./07-devops-infrastructure/02-infrastructure-as-code.md) | Terraform, drift detection, immutable infra |
| 3 | [Containers & Orchestration](./07-devops-infrastructure/03-containers-orchestration.md) | Docker, Kubernetes, Helm |

---

## Quick Reference: When to Use What

```
Choosing an architecture pattern?
├── Small team, simple domain → Layered Architecture
├── Need testability above all → Hexagonal Architecture
├── Complex business rules → Clean Architecture
├── Independent deployability → Microservices
├── High write/read ratio difference → CQRS
└── Need full audit trail → Event Sourcing

Choosing a design pattern?
├── Object creation is complex → Creational patterns
├── Classes need to work together → Structural patterns
└── Objects need to communicate → Behavioral patterns
```

---

## Conventions

- **Language:** Code examples use Python unless a concept is language-specific.
- **Diagrams:** All diagrams use [Mermaid](https://mermaid.js.org/) syntax.
- **Anti-patterns:** Every topic shows what breaks *before* the solution.
- **Trade-offs:** Every topic has a "When NOT to Use" section.
