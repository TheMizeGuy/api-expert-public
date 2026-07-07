# Inter-Service Communication

## Service mesh (2026)

### Why use one

Handle mTLS, traffic shaping, retries, circuit breakers, and observability at infrastructure layer.

### Mesh comparison

| Mesh | Data plane | Overhead | Strength |
|---|---|---|---|
| **Istio** | Envoy (sidecar) or ztunnel+waypoint (ambient) | Ambient: 3-15% | Most capable, CNCF graduated 2023 |
| **Linkerd** | linkerd2-proxy (Rust) | ~10 MB/sidecar | Lowest overhead, simplest ops |
| **Consul Connect** | Envoy | ~30 MB/sidecar | HashiCorp ecosystem, k8s + VMs |
| **Cilium** | eBPF (no sidecar) | Minimal | Kernel-space routing, no sidecar |

**Istio ambient mode**: ztunnel DaemonSet for L4 mTLS, waypoint proxies only where L7 needed. Eliminates sidecar overhead.

## API gateways

| Gateway | Sweet spot |
|---|---|
| **Kong** | General-purpose API management with plugins |
| **KrakenD** | BFF aggregation (70K+ req/s, stateless Go) |
| **Envoy Gateway** | Cloud-native, k8s Gateway API conformance |
| **Traefik** | Dev-friendly k8s ingress, auto-discovery |
| **Apollo Router** | GraphQL federation (supergraph) |

## Resilience patterns

### Circuit breaker

States: Closed (normal) -> Open (failing, reject requests) -> Half-Open (test recovery).

| Library | Language |
|---|---|
| **Resilience4j** | Java/JVM |
| **Polly** | .NET |
| **opossum** | Node.js |
| **Envoy outlier detection** | Any (infrastructure-level) |

### Retry with backoff

- Exponential backoff with **decorrelated jitter** (AWS Builders recommendation)
- Set retry budget (max 20% of requests can be retries)
- Propagate deadlines across services (gRPC deadline propagation)

### Request hedging

Send duplicate to second replica at P80 latency delta. AWS case study: 29% p99 reduction, 8% duplicate rate.

### Bulkhead isolation

Separate thread/connection pools per downstream service. Failure in one doesn't cascade.

## Idempotency keys (Stripe pattern)

| Step | Detail |
|---|---|
| 1 | Client sends `Idempotency-Key: <uuid>` on POST/PATCH |
| 2 | Server checks if key exists in store |
| 3 | If exists: return cached response (200, not 409) |
| 4 | If new: process request, store key + response (24h TTL) |

## Message brokers (2026)

| Broker | Strength | Key change |
|---|---|---|
| **Kafka** | High-throughput event streaming, durable log | KRaft mode only (ZooKeeper removed in 4.0) |
| **RabbitMQ** | Traditional message queue, flexible routing | Khepri metadata store (4.2), AMQP 1.0 default |
| **NATS** | Ultra-low latency, simple pub/sub | JetStream for persistence + exactly-once |
| **Redis Streams** | Lightweight, familiar Redis interface | Good for moderate throughput |

### When to pick which

| Use case | Broker |
|---|---|
| Event streaming, replay, large-scale | Kafka |
| Task queue, request-reply, flexible routing | RabbitMQ |
| Ultra-low latency, simple pub/sub | NATS |
| Lightweight, already have Redis | Redis Streams |

## Saga pattern

Orchestrate multi-service transactions where each step can be compensated (rolled back).

| Variant | Coordinator | Best for |
|---|---|---|
| **Orchestration** (Temporal, Step Functions) | Central workflow engine | Complex flows, visibility |
| **Choreography** | Events between services | Simple flows, loose coupling |

**Temporal** is the 2026 reference implementation for saga orchestration.

## Transactional outbox + CDC

Solve dual-write problem (write to DB + publish event atomically):
1. Write event to `outbox` table in same DB transaction as business data
2. CDC (Debezium) tails the outbox table's WAL
3. CDC publishes to Kafka/RabbitMQ

Guarantees at-least-once delivery without distributed transactions.

## Event sourcing + CQRS

- **Event sourcing**: store events, not current state. Rebuild state by replaying.
- **CQRS**: separate read and write models. Write model = event store. Read model = materialized projections.
- Use when: audit trail required, temporal queries, complex domain logic.

## GraphQL federation

Apollo Router composes multiple GraphQL subgraphs into one supergraph. Each team owns their subgraph. `@key`, `@external`, `@requires` directives for cross-subgraph references.
