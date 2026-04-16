---
name: migrate-api
description: |-
  Migrate or refactor APIs — version upgrades (v1→v2), protocol changes (REST→GraphQL, REST→gRPC, REST→Connect-RPC), framework migrations (Express→Hono, Flask→FastAPI), schema restructuring, cross-project consolidation (split one API into two, merge two into one). Uses dual-read + shadow-traffic + blue-green patterns to minimize risk. Maintains backward compatibility via Stripe-style evolutionary versioning or version-rewriting middleware. Triggers on "migrate from X to Y", "upgrade v1 to v2", "move from rest to graphql", "refactor across repos", "split my api", "merge these apis", "consolidate api services". Produces migration plan + runnable incremental steps + rollback procedure.
argument-hint: '<source format/version> to <target format/version>'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
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

## Step 2: Dispatch api-expert

```
Agent({
  description: "API migrate: <from> to <to>",
  subagent_type: "api-expert:api-expert",
  model: "opus",
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

Read ${CLAUDE_PLUGIN_ROOT}/references/schema-design.md and documentation-lifecycle.md FIRST, plus the target-protocol reference file.

DELIVERABLES:
1. Catalog current state (every affected endpoint + contract shape)
2. Design target state (endpoint mapping, field-level mapping, breaking vs non-breaking)
3. Migration pattern selection (Stripe evolutionary / URL versioning / header versioning / parallel services / dual-read / shadow traffic / blue-green / strangler fig)
4. Incremental migration plan (ordered steps, each independently reversible)
5. Compatibility layer (if needed)
6. Consumer communication plan
7. Rollback procedures (per-step + emergency full revert)
8. Testing strategy (contract tests for both source + target during overlap)
9. Observability during migration (old vs new endpoint rate, mismatch rate)
10. Timeline with gates

CONSTRAINTS:
- NEVER break behavioral contracts without deprecation + sunset headers
- GREP all external consumers before changing anything
- PRESERVE operationIds, GraphQL field names, Protobuf field numbers
- PREFER adding new alongside existing over replacing

PROCEED.
```

## Step 3: Relay the plan

Present the catalog + pattern selection + incremental steps. User reviews + approves steps before execution.

## Never do

- Change external-facing contracts without deprecation flow
- Migrate in one big step — always incremental
- Skip the shadow traffic / dual-read phase for high-traffic migrations
- Delete old code on day 1 of cutover — keep it quiescent for at least one full release cycle
