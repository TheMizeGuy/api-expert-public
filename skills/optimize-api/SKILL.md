---
name: optimize-api
description: |-
  Optimize API performance and scalability. Measures first (pg_stat_statements, OpenTelemetry traces, k6 profile, CPU/memory metrics), identifies top bottlenecks by impact × effort, then applies evidence-backed optimizations — query tuning, indexes, connection pooling (PgBouncer/pgcat), caching (Redis L2 + in-process L1), N+1 elimination (DataLoader), cache stampede prevention (XFetch/single-flight), rate limiting tuning, serialization (protobuf over JSON), HTTP/2-3, CDN edge caching, tail latency reduction (request hedging P80). Triggers on "optimize my api", "api is slow", "reduce latency", "handle more traffic", "scale my api", "reduce 5xx rate", "improve throughput". Produces before/after metrics for every change.
argument-hint: '<target — endpoint, SLO, or bottleneck you want addressed>'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
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
  model: "opus",
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

MEASURE FIRST — NEVER OPTIMIZE BLIND.

DELIVERABLES:
1. Measurement baseline
2. Top 3-5 bottlenecks by impact x effort
3. Proposed changes (in priority order), each with: change description + code diff, expected impact, effort estimate (S/M/L), verification method
4. Application plan — sequence of changes with measurement between each
5. Rollback plan for each change
6. Long-term recommendations (if any require broader work)

PROCEED.
```

## Step 3: Relay the plan

Present the bottleneck ranking + proposed changes + expected impact. Confirm with user before applying.

## Never do

- Optimize without measuring first (premature optimization)
- Apply multiple changes in one commit (can't isolate impact)
- Change behavioral contracts under the guise of optimization
- Suggest rewriting from scratch when incremental fixes work
- Skip measuring after-state — unverified optimizations rot into folklore
