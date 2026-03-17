# Operations Analysis

Performance, resource utilization, failure modes, scalability, and operability analysis for technical specifications.

> **Part of the spec-review suite.** Can run standalone (performs its own discovery) or as a sub-command of `/pmp:spec-review` (discovery already done). When standalone, first execute [discovery.md](../../spec-review/references/discovery.md).

## Standalone Mode

When invoked directly (not dispatched by orchestrator):
1. Read [discovery.md](../../spec-review/references/discovery.md) and execute Phase 0 + Phase 1
2. Proceed to the analysis phases below
3. Produce a standalone report using [spec-operations-output.md](../assets/spec-operations-output.md)

When dispatched by `/pmp:spec-review`, skip directly to the analysis phases — the system model is already in context.

---

### Phase 8: Performance & Architectural Optimization

Construct the **critical request path**. Identify:
- synchronous dependencies
- blocking operations
- network round trips
- serialization costs
- policy engine latency
- logging overhead

Evaluate potential impact on: P50, P95, P99 latency and throughput.

#### Optimization Analysis

Beyond identifying bottlenecks, evaluate the design for:
- unnecessary network hops between components
- avoidable serialization/deserialization (data converted between formats without need)
- excessive abstractions that add latency without value
- redundant processing stages (same data processed multiple times)
- cross-module dependencies that force synchronous coordination

Suggest optimizations that:
- reduce critical path latency
- minimize state that must be shared or synchronized
- reduce cross-module dependencies
- simplify data flow through the system
- eliminate unnecessary indirection

---

### Phase 9: Resource Utilization

Analyze resource consumption risks:
- CPU usage, memory allocation
- goroutine/thread growth
- connection pool exhaustion
- file descriptor limits
- queue buildup

Identify any: unbounded buffers, unbounded concurrency, memory leak risks, backpressure gaps.

---

### Phase 10: Failure Mode Analysis

Simulate failures:
- upstream outage
- policy engine crash
- network partition
- slow client
- logging pipeline failure
- configuration corruption

Check whether the system: fails open, fails closed, creates retry storms, causes cascading failures.

Ensure the architecture defines:
- safe fallback behavior for every failure mode
- deterministic error propagation (errors don't silently change behavior)
- graceful degradation paths

---

### Phase 11: Scalability Analysis

Evaluate scaling behavior under:
- high request concurrency
- multi-tenant deployments
- large policy sets
- multi-region operation

Identify bottlenecks: centralized state, synchronous control plane calls, shared caches, global locks.

---

### Phase 12: Operability

Evaluate operational viability:

**Observability:** metrics, distributed tracing, request correlation, policy decision logs, debugging visibility.

Verify the system can answer:
- What happened? (structured logging, event trail)
- Why did it happen? (decision logs, causal tracing)
- What decision was made? (policy evaluation audit)
- What policy was applied? (configuration audit trail)

**Lifecycle:** configuration management, rollout strategy, backward compatibility, schema evolution, upgrade paths.
