# 04 — Distributed Systems

> The hard problems that appear when services run on multiple machines and communicate over an unreliable network.

Distributed systems introduce a new class of failures: network partitions, partial failures, clock skew, and consistency trade-offs that simply don't exist in a single-process system. Understanding these problems is essential for building systems that behave correctly under real-world conditions.

## Topics

| # | Topic | Core Problem |
|---|-------|--------------|
| 1 | [CAP Theorem](./01-cap-theorem.md) | You can't have consistency AND availability during a partition |
| 2 | [Consistency Models](./02-consistency-models.md) | What guarantees can callers rely on across replicas? |
| 3 | [Consensus Algorithms](./03-consensus-algorithms.md) | How do nodes agree on a single value? |
| 4 | [Distributed Transactions](./04-distributed-transactions.md) | How do multi-service operations stay atomic? |
| 5 | [Service Mesh](./05-service-mesh.md) | How do services talk to each other securely and reliably? |
| 6 | [Circuit Breaker & Resilience](./06-circuit-breaker-resilience.md) | How does a system survive when a dependency fails? |

## The Fallacies of Distributed Computing

Peter Deutsch's eight assumptions newcomers make — and why they're wrong:

1. The network is reliable (it's not — packets drop)
2. Latency is zero (it's not — cross-datacenter is 10–100ms)
3. Bandwidth is infinite (it's not)
4. The network is secure (it's not)
5. Topology doesn't change (it does — nodes come and go)
6. There is one administrator (there isn't)
7. Transport cost is zero (it's not)
8. The network is homogeneous (it's not)

Build with these realities in mind, not against them.
