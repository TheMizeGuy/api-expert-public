# Architecture Patterns and Protocols

## Architectural styles

| Style | Team size | Data store | Use when |
|---|---|---|---|
| **Monolith** | 1-10 | Single shared DB | Product < 12 months, unclear domain boundaries |
| **Modular monolith** | 10-100 | One DB, schema-per-module | Visible domain boundaries, want independent dev velocity without distributed ops |
| **Microservices** | 100+ | One DB per service | Genuinely independent scaling needs, polyglot requirements, regulatory isolation |
| **Macroservices** | 20-200 | Per-service ownership | Want independent deploy without 200-service tax |

**2026 consensus**: 42% of orgs consolidating microservices back into larger units. Modular monolith is the default for new services. Amazon Prime Video reversed microservices for 90% cost reduction.

## Protocol decision matrix

| Protocol | Best for | Encoding | Codegen |
|---|---|---|---|
| **REST + OpenAPI 3.1** | External APIs, SDK generation, broad compatibility | JSON | Stainless, Speakeasy, Fern, Kiota |
| **GraphQL** | Client-driven multi-service composition, BFF | JSON | graphql-codegen, Apollo Rover |
| **gRPC + Protobuf** | Internal high-perf service-to-service, polyglot | Binary varint | protoc, buf, ts-proto, connect-rpc |
| **tRPC** | TypeScript monorepo, same-team frontend + backend | JSON | Built-in (TS inference) |
| **Connect-RPC** | gRPC semantics + browser-native HTTP | Binary or JSON | @connectrpc/connect |

**Default 2026 recommendation**: REST + OpenAPI 3.1 for external; gRPC/Connect for internal; GraphQL for client-driven composition.

## API Gateway vs BFF

| Pattern | Intent | When |
|---|---|---|
| **API Gateway** | Single entry point: auth, rate limiting, routing | General external traffic |
| **BFF (Backend for Frontend)** | One backend per client tier (web/iOS/Android) | Different clients need different aggregations |

**BFF is 2026 best practice for browser apps** — tokens never in browser. IETF draft-ietf-oauth-browser-based-apps recommends.

## Gateway comparison

| Gateway | Architecture | Sweet spot |
|---|---|---|
| **Kong** | Nginx-embedded, plugin ecosystem | General-purpose API management |
| **KrakenD** | Stateless Go, purpose-built aggregation | BFF aggregation (70K+ req/s) |
| **Envoy Gateway** | C++, xDS, k8s Gateway API | Cloud-native, highest Gateway API conformance |
| **Traefik** | Go, auto-service-discovery | Dev-friendly k8s ingress |

## Real-time patterns

| Pattern | Latency | Complexity | Use when |
|---|---|---|---|
| **SSE** | ~100ms | Low | Server-to-client only (dashboards, feeds) |
| **WebSocket** | ~50ms | Medium | Bidirectional (chat, collaboration) |
| **Long polling** | ~1s | Low | Fallback for environments blocking WS |
| **gRPC streaming** | ~10ms | Medium | Service-to-service real-time |

## HTTP versions

| Version | Key improvement | Deploy note |
|---|---|---|
| HTTP/1.1 | Keep-alive, pipelining | Universal baseline |
| HTTP/2 | Multiplexing, header compression, server push | Default on modern load balancers |
| HTTP/3 (QUIC) | UDP-based, 0-RTT, loss-resilient | Good for mobile/lossy networks; CDN support growing |

## DDD fundamentals

- **Bounded context**: area where a single domain model is consistent — maps to service boundary
- **Aggregate**: cluster of domain objects treated as one consistency unit
- **Event storming**: collaborative workshop to discover bounded contexts BEFORE choosing boundaries

## Common regrets

| Regret | Root cause | Fix |
|---|---|---|
| "47 microservices for 8 engineers" | Architecture from blog posts, not requirements | Collapse to modular monolith |
| "Every deploy coordinates 6 services" | Distributed monolith (shared schemas) | Strict versioning + backward compatibility per service |
| "90% of request is network + serialization" | N service hops per user request | API composition at gateway; cache aggregates |
