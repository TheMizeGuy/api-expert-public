---
name: design-api
description: |-
  Design a new API from scratch. Use when the user wants to build a new API service and needs architectural guidance on protocol choice (REST vs GraphQL vs gRPC vs tRPC vs Connect), schema design, authentication (OAuth 2.1/OIDC/JWT/session/BFF), authorization (RBAC/ABAC/ReBAC), error format (RFC 9457), pagination, rate limiting, observability, and deployment. Triggers on "design an api", "build a new api", "api for X", "what's the best way to structure an api for X", "help me design an API", "new api service". Produces a complete design document with OpenAPI/GraphQL/Protobuf spec, runnable scaffold in the user's preferred stack, deployment plan, and testing strategy.
argument-hint: '<what the API should do — domain, clients, expected scale>'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
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

Ask 1-3 of these at most — don't interrogate. If user says "you pick", proceed with sensible defaults (modular monolith, REST + OpenAPI 3.1, TypeScript).

## Step 2: Dispatch api-expert

```
Agent({
  description: "API design: <domain>",
  subagent_type: "api-expert:api-expert",
  model: "opus",
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
1. Architectural decision record (protocol, monolith vs microservices, auth, authz, DB, caching, deployment)
2. Complete API specification (OpenAPI 3.1 YAML / GraphQL SDL / .proto file)
3. Runnable scaffold (framework setup, auth middleware, error handler, rate limiting, health check, pagination, idempotency, OTel, structured logging)
4. Testing plan (unit, contract, load, security)
5. Deployment plan
6. Observability plan (structured log fields, key metrics, alert rules)
7. Lifecycle plan (versioning, deprecation, changelog)

CONSTRAINTS:
- Match existing project conventions if a codebase is present
- Preserve behavioral contracts on any existing endpoints
- Use project's existing package manager (check lockfile)
- Prefer proven libraries over bleeding-edge

PROCEED.
```

## Step 3: Relay the design

Present the Summary + Architectural Decision Record to the user first. Offer to drill into the spec, scaffold, or deployment plan on request.

## Never do

- Suggest a stack the user didn't ask for without justification
- Skip the OpenAPI/GraphQL/Protobuf spec deliverable
- Produce vague "should consider" advice — always land on a concrete recommendation
- Add emojis
