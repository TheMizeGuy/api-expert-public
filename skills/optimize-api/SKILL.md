---
name: optimize-api
description: |-
  Optimize API performance and scalability. Measures first (pg_stat_statements, OpenTelemetry traces, k6 profile, CPU/memory metrics), identifies top bottlenecks by impact × effort, then applies evidence-backed optimizations — query tuning, indexes, connection pooling (PgBouncer/pgcat), caching (Redis L2 + in-process L1), N+1 elimination (DataLoader), cache stampede prevention (XFetch/single-flight), rate limiting tuning, serialization (protobuf over JSON), HTTP/2-3, CDN edge caching, tail latency reduction (request hedging P80). Triggers on "optimize my api", "api is slow", "reduce latency", "handle more traffic", "scale my api", "reduce 5xx rate", "improve throughput". Produces before/after metrics for every change.
argument-hint: '<target — endpoint, SLO, or bottleneck you want addressed>'
allowed-tools: Agent, Read, Grep, Glob, Bash, Edit, Write, TodoWrite, WebSearch, WebFetch, mcp__goodmem__goodmem_memories_retrieve, mcp__goodmem__goodmem_memories_get, mcp__goodmem__goodmem_memories_create, mcp__context7__resolve-library-id, mcp__context7__query-docs, mcp__plugin_serena_serena__activate_project, mcp__plugin_serena_serena__get_symbols_overview, mcp__plugin_serena_serena__find_symbol, mcp__plugin_serena_serena__find_referencing_symbols, mcp__plugin_serena_serena__list_dir, mcp__plugin_serena_serena__search_for_pattern
---

# Optimize API

Dispatches the api-expert agent with a performance-optimization briefing. Measure first, then fix.

## Step 1: Gather measurement context

Before dispatching, establish:

| Info | Why |
|---|---|
| Current performance | Baseline |
| Target SLO (p95 latency, error rate) | Goal |
| Traffic level (RPS, concurrent users) | Scale context |
| Worst-performing endpoint / operation | Focus area |
| Bottleneck hypothesis (if any) | Seed for investigation |
| Access to pg_stat_statements / OTel traces / metrics dashboards? | Evidence source |

## Step 2: Dispatch api-expert

```
Agent({
  description: "API optimize: <target>",
  subagent_type: "api-expert:api-expert",
  // model omitted — inherits the session model (always the strongest available Claude)
  prompt: "<see briefing below>"
})
```

### Briefing

```
ORIGINAL USER REQUEST: <verbatim>

WORKFLOW: optimize

CURRENT STATE: <baseline metrics>
TARGET SLO: <goal>
TRAFFIC: <current + target RPS>
FOCUS ENDPOINT(S): <list>
EVIDENCE ACCESS: <pg_stat_statements / OTel / dashboards / none>

Read ${CLAUDE_PLUGIN_ROOT}/references/performance-caching.md FIRST, then inter-service.md and observability.md.

MEASURE FIRST — NEVER OPTIMIZE BLIND:

Step 1 — Identify top bottlenecks
- If pg_stat_statements available: ORDER BY total_exec_time DESC LIMIT 10
- If OTel available: analyze p95/p99 latency by span name + service
- If k6 available: profile under target load
- If logs only: grep for slow request patterns, count by endpoint

Step 2 — Rank by impact × effort
  - High impact + Low effort = fix first
  - High impact + High effort = plan + schedule
  - Low impact = skip unless cheap

Step 3 — Apply evidence-backed fixes by category:

  Database:
  - Missing indexes (EXPLAIN ANALYZE the slow query)
  - N+1 queries (DataLoader / Prisma include / Drizzle with)
  - Connection pool sizing (HikariCP formula: ((core×2) + spindles))
  - PgBouncer transaction mode (protocol-level prepared statements in 1.21+)
  - Statement timeout + lock timeout + idle_in_transaction_session_timeout
  - Materialized views for aggregate queries (identify via pg_stat_statements)

  Caching:
  - HTTP caching (Cache-Control public, s-maxage, stale-while-revalidate, ETag)
  - CDN edge caching (Cloudflare Tiered Cache, Fastly, Vercel)
  - Application L1 (lru-cache) + L2 (Redis) tiered
  - Cache stampede prevention (XFetch probabilistic early expiration OR single-flight)
  - Query result caching (Prisma query cache, Drizzle query cache)
  - Cache key design (deterministic, sorted params, version prefix)

  Rate limiting:
  - Token bucket / GCRA tuning
  - Redis Lua for atomic ops
  - Per-tenant quotas aligned with SLO tiers

  Serialization:
  - Protobuf over JSON for high-throughput internal services
  - zstd / brotli for large payloads
  - HTTP/2 multiplexing, HTTP/3 QUIC for mobile + lossy networks

  Service-to-service:
  - Circuit breaker (Envoy outlier detection / Resilience4j / Polly)
  - Request hedging at P80 delta (29% p99 reduction, 8% duplicate rate — AWS case study)
  - Retry with exponential backoff + decorrelated jitter
  - gRPC deadlines propagate across services
  - Bulkhead isolation per downstream

  Infrastructure:
  - Horizontal replicas per region
  - Vertical scaling if connection-bound
  - Disable sleep/idle-scale-to-zero for low-latency services

Step 4 — Measure after each change
- Confirm improvement before moving to next optimization
- Rollback if metric regresses
- Document before/after numbers

Step 5 — Document gotchas specific to this service
- If goodmem is configured, write a learning if the optimization uncovered a non-obvious pattern

DELIVERABLES:
1. Measurement baseline
2. Top 3-5 bottlenecks by impact × effort
3. Proposed changes (in priority order), each with:
   - Change description + code diff
   - Expected impact (latency / throughput / error rate)
   - Effort estimate (S/M/L)
   - Verification method
4. Application plan — sequence of changes with measurement between each
5. Rollback plan for each change
6. Long-term recommendations (if any require broader work)

PROCEED. Read the reference files above, query goodmem if configured, then measure + optimize.
```

## Step 3: Verify, then relay the plan

Before presenting, verify the returned plan: a measurement baseline exists (no baseline = the plan
is guesswork, reject it); every proposed change carries expected impact, effort (S/M/L), a
verification method, and a rollback plan. Missing pieces → re-query the agent before relaying.

Present the bottleneck ranking + proposed changes + expected impact. Confirm with user before applying. Offer to apply changes one-by-one with measurement between each.

## Execution mode

The dispatched agent inherits the session model — always the strongest available Claude, never a pinned or dated model. If the session model is already the strongest tier and the task is important or complicated, this skill may run the optimization workflow inline in the main context instead of dispatching a separate agent. Never block on, or wait for, a model that isn't the session model.

## Never do

- Optimize without measuring first (premature optimization)
- Apply multiple changes in one commit (can't isolate impact)
- Change behavioral contracts (error strings, status codes, log/metric names) under the guise of optimization
- Suggest rewriting from scratch when incremental fixes work
- Skip measuring after-state — unverified optimizations rot into folklore
