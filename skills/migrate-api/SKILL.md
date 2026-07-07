---
name: migrate-api
description: |-
  Migrate or refactor APIs — version upgrades (v1→v2), protocol changes (REST→GraphQL, REST→gRPC, REST→Connect-RPC), framework migrations (Express→Hono, Flask→FastAPI), schema restructuring, cross-project consolidation (split one API into two, merge two into one). Uses dual-read + shadow-traffic + blue-green patterns to minimize risk. Maintains backward compatibility via Stripe-style evolutionary versioning or version-rewriting middleware. Triggers on "migrate from X to Y", "upgrade v1 to v2", "move from rest to graphql", "refactor across repos", "split my api", "merge these apis", "consolidate api services". Produces migration plan + runnable incremental steps + rollback procedure.
argument-hint: '<source format/version> to <target format/version>'
allowed-tools: Agent, Read, Grep, Glob, Bash, Edit, Write, TodoWrite, WebSearch, WebFetch, mcp__goodmem__goodmem_memories_retrieve, mcp__goodmem__goodmem_memories_get, mcp__goodmem__goodmem_memories_create, mcp__context7__resolve-library-id, mcp__context7__query-docs, mcp__plugin_serena_serena__activate_project, mcp__plugin_serena_serena__get_symbols_overview, mcp__plugin_serena_serena__find_symbol, mcp__plugin_serena_serena__find_referencing_symbols, mcp__plugin_serena_serena__list_dir, mcp__plugin_serena_serena__search_for_pattern
---

# Migrate API

Dispatches the api-expert agent with a migration briefing. Migrations are high-risk — always incremental, always reversible.

## Step 1: Scope the migration

| Info | Why |
|---|---|
| Source state | Baseline |
| Target state | Goal |
| Number of affected endpoints | Effort estimate |
| Known consumers (internal + external) | Cutover strategy |
| SLA / uptime requirements | Determines blue-green vs dual-read complexity |
| Deadline / budget | Timeline pressure |

## Step 2: Dispatch api-expert

```
Agent({
  description: "API migrate: <from> → <to>",
  subagent_type: "api-expert:api-expert",
  // model omitted — inherits the session model (always the strongest available Claude)
  prompt: "<see briefing below>"
})
```

### Briefing

```
ORIGINAL USER REQUEST: <verbatim>

WORKFLOW: migration

SOURCE: <version / protocol / framework / repo>
TARGET: <version / protocol / framework / repo>
AFFECTED ENDPOINTS: <list or estimate>
KNOWN CONSUMERS: <internal services + external clients>
SLA: <uptime target>
DEADLINE: <if any>

Read ${CLAUDE_PLUGIN_ROOT}/references/schema-design.md and documentation-lifecycle.md FIRST, plus the target-protocol reference file (architecture-patterns.md or inter-service.md depending).

DELIVERABLES:

1. Catalog current state
   - List every affected endpoint with its contract shape
   - Identify shared types / models
   - Identify shared middleware (auth, rate limit, logging)
   - Identify consumers (grep external repos, check SDK download counts); tag each consumer with
     its evidence source — `grep:<path>` (show the command) or `assumed` — and list any repo you
     could not access

2. Design target state
   - Explicit mapping: current endpoint → target endpoint (or deprecation)
   - Field-level mapping for payload changes
   - Breaking vs non-breaking classification per endpoint
   - Cutover sequence (bottom-up: shared code → individual services → external surface)

3. Migration pattern selection (pick the right one):
   - **Stripe evolutionary**: version-rewriting middleware translates new canonical shape → old client's pinned version. Zero breaking for external clients, complex internally.
   - **URL versioning (v1/ → v2/)**: simplest, explicit, but fragments codebase and consumer SDKs
   - **Header versioning (Accept-Version)**: transparent to URLs, allows gradual migration
   - **Parallel services**: new service alongside old, gradual traffic shift via gateway/mesh
   - **Dual-read**: new code reads from v1 AND v2 data paths, compares results, reports mismatches (validation phase before cutover)
   - **Shadow traffic**: mirror prod traffic to new version, measure correctness/performance without user impact
   - **Blue-green**: atomic cutover at gateway layer after full validation
   - **Strangler fig**: new service proxies unhandled requests to old; grows coverage over time

4. Incremental migration plan (ordered steps):
   - Step 1: <what, verification, rollback>
   - Step 2: ...
   - Each step must be independently reversible
   - Each step must have explicit verification before proceeding

5. Compatibility layer (if needed):
   - Write request-rewriting middleware OR translator service
   - Cover all breaking changes in this layer
   - Fixed maintenance cost via tight encapsulation (Stripe's pattern)

6. Consumer communication plan:
   - Who to notify (internal + external)
   - Channels (email, dashboard, changelog, blog post)
   - Timeline (12-24 month external deprecation windows typical)
   - Migration guide
   - Sunset date

7. Rollback procedures:
   - Per-step rollback instructions
   - Emergency rollback plan (full revert in <15 min)
   - Data rollback if schema changed
   - Traffic rollback at gateway

8. Testing strategy:
   - Contract tests pinned to source state before migration
   - Add contract tests for target state during migration
   - Dual-test both APIs during overlap period
   - Load test target state before cutover
   - Security audit target state

9. Observability during migration:
   - Extra metrics: old-endpoint call rate, new-endpoint call rate, mismatch rate (if dual-read)
   - Error rate alert thresholds tightened during migration
   - Dashboards comparing old vs new

10. Timeline with gates:
    - Gate 1: Target state deployed, shadow traffic for N days
    - Gate 2: Dual-read validation passes
    - Gate 3: X% traffic on new path for N days
    - Gate 4: 100% traffic, old path standby
    - Gate 5: Old path removed (after grace period)

CONSTRAINTS:
- NEVER break behavioral contracts without explicit deprecation + sunset headers (RFC 9745 + RFC 8594)
- NEVER change error strings, status codes, log field names, metric names without propagating to callers
- GREP all external consumers before changing anything — surprise breakages destroy trust
- PRESERVE operationIds, GraphQL field names, Protobuf field numbers even when renaming conceptually (add deprecation instead)
- PREFER adding new alongside existing over replacing

PROCEED. Read the reference files above, query goodmem for prior migration learnings if configured, then produce the plan.
```

## Step 3: Verify, then relay the plan

Before presenting, verify the returned plan: the pattern selection states why it beats the
alternatives for THIS migration; every incremental step has its own verification and rollback; the
consumer catalog is grounded in grep evidence, not assumption. Missing pieces → re-query the agent
before relaying.

Present the catalog + pattern selection + incremental steps. User reviews + approves steps before execution. Offer to execute the first step (with rollback prepared).

## Execution mode

The dispatched agent inherits the session model — always the strongest available Claude, never a pinned or dated model. If the session model is already the strongest tier and the task is important or complicated, this skill may run the migration workflow inline in the main context instead of dispatching a separate agent. Never block on, or wait for, a model that isn't the session model.

## Never do

- Change external-facing contracts without deprecation flow
- Migrate in one big step — always incremental
- Skip the shadow traffic / dual-read phase for high-traffic migrations
- Assume internal consumers can change in lockstep (they usually can't)
- Delete old code on day 1 of cutover — keep it quiescent for at least one full release cycle
