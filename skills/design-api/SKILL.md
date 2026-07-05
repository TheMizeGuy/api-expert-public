---
name: design-api
description: |-
  Design a new API from scratch. Use when the user wants to build a new API service and needs architectural guidance on protocol choice (REST vs GraphQL vs gRPC vs tRPC vs Connect), schema design, authentication (OAuth 2.1/OIDC/JWT/session/BFF), authorization (RBAC/ABAC/ReBAC), error format (RFC 9457), pagination, rate limiting, observability, and deployment. Triggers on "design an api", "build a new api", "api for X", "what's the best way to structure an api for X", "help me design an API", "new api service". Produces a complete design document with OpenAPI/GraphQL/Protobuf spec, runnable scaffold in the user's preferred stack, deployment plan, and testing strategy.
argument-hint: '<what the API should do — domain, clients, expected scale>'
allowed-tools: Agent, Read, Grep, Glob, Bash, Write, TodoWrite, WebSearch, WebFetch, mcp__goodmem__goodmem_memories_retrieve, mcp__goodmem__goodmem_memories_get, mcp__context7__resolve-library-id, mcp__context7__query-docs, mcp__plugin_serena_serena__activate_project, mcp__plugin_serena_serena__list_dir
---

# Design API

Dispatches the api-expert agent with a design-workflow briefing.

## Step 1: Gather requirements

Before dispatching, collect from the user if missing:

| Question | Why it matters |
|---|---|
| What does the API do? (domain, core resources) | Determines schema shape |
| Who calls it? (web / mobile / CLI / third-party devs) | Determines BFF need, SDK gen need, auth model |
| What's the expected scale? (RPS, concurrent users) | Determines caching/rate-limit/scaling needs |
| Multi-tenant? | Triggers tenant isolation patterns |
| Real-time requirements? (WebSocket/SSE/polling) | Protocol choice |
| Language/stack preferences? (TypeScript/Python/Go/Rust/Swift) | Framework selection |
| Deployment target? | Ops patterns |
| Budget/timeline? | Tradeoff guidance |

Ask 1-3 of these at most — don't interrogate. If user says "you pick", proceed with sensible defaults (modular monolith, REST + OpenAPI 3.1, TypeScript).

## Step 2: Dispatch api-expert

```
Agent({
  description: "API design: <domain>",
  subagent_type: "api-expert:api-expert",
  // model omitted — inherits the session model (always the strongest available Claude)
  prompt: "<see briefing below>"
})
```

### Briefing

```
ORIGINAL USER REQUEST: <verbatim>

WORKFLOW: design

REQUIREMENTS:
- Domain: <what the API does>
- Clients: <list>
- Scale: <RPS estimate>
- Multi-tenant: <yes/no, isolation model if known>
- Real-time: <yes/no, pattern if known>
- Stack: <language/framework>
- Deployment: <target>
- Other constraints: <any>

Read ${CLAUDE_PLUGIN_ROOT}/references/architecture-patterns.md FIRST, then schema-design.md, authentication.md, authorization.md.

DELIVERABLES (produce all of these):
1. Architectural decision record:
   - Protocol choice (REST/GraphQL/gRPC/tRPC/Connect) with rationale
   - Monolith vs microservices decision with rationale
   - Auth model (OAuth/OIDC/session/BFF) with rationale
   - Authorization model (RBAC/ABAC/ReBAC) with rationale
   - Database + caching strategy
   - Deployment architecture
2. Complete API specification (OpenAPI 3.1 YAML / GraphQL SDL / .proto file depending on protocol choice)
3. Runnable scaffold — file tree + key source files showing:
   - Framework setup (Hono/Fastify/Express/FastAPI/etc.)
   - Auth middleware wired up
   - Error handler producing RFC 9457 Problem JSON
   - Rate limiting middleware
   - Health check endpoint
   - OpenAPI doc serving route
   - Pagination contract (cursor-based default)
   - Idempotency-Key middleware (Stripe pattern)
   - OTel instrumentation
   - Structured logging (pino/structlog)
4. Testing plan:
   - Unit test scaffold
   - Contract tests (Pact) or spec validation (Schemathesis)
   - Load test (k6) with SLO-aligned thresholds
   - Security scan (ZAP baseline)
5. Deployment plan:
   - Deployment config for the target platform, with healthcheck + restart policy
   - Required env vars
   - Migration strategy if DB
6. Observability plan:
   - Structured log fields
   - Key metrics (RED method for API + USE method for resources)
   - Alert rules (multi-window multi-burn-rate on SLOs)
7. Lifecycle plan:
   - Versioning strategy (URL-path / header / evolutionary — pick per scale)
   - Deprecation policy
   - Changelog format

CONSTRAINTS:
- Match existing project conventions if a codebase is present (grep first)
- Preserve behavioral contracts on any existing endpoints
- Use project's existing package manager (check lockfile)
- Prefer proven libraries over bleeding-edge

PROCEED with your standard workflow (reference files first, then goodmem if configured, then context7, then deliver).
```

## Step 3: Verify, then relay the design

Before presenting, verify the returned design: all 7 numbered deliverables present; every ADR entry
states a rationale (a choice without a why is incomplete); the spec is complete, not a fragment.
Missing pieces → re-query the agent before relaying.

Present the Summary + Architectural Decision Record to the user first. Offer to drill into the spec, scaffold, or deployment plan on request. If the user wants to proceed to implementation, offer to dispatch the agent again in "create" mode (edit tools enabled) to scaffold the actual codebase.

## Execution mode

The dispatched agent inherits the session model — always the strongest available Claude, never a pinned or dated model. If the session model is already the strongest tier and the task is important or complicated, this skill may run the design workflow inline in the main context instead of dispatching a separate agent. Never block on, or wait for, a model that isn't the session model.

## Never do

- Suggest a stack the user didn't ask for without justification
- Skip the OpenAPI/GraphQL/Protobuf spec deliverable
- Produce vague "should consider" advice — always land on a concrete recommendation
- Add emojis
