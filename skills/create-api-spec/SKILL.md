---
name: create-api-spec
description: |-
  Generate or update API specifications — OpenAPI 3.1 YAML, GraphQL SDL, or Protocol Buffers (.proto). Supports contract-first (write spec, generate code) and code-first (extract spec from existing code). Adds proper error schemas (RFC 9457 Problem JSON), pagination contracts (cursor-based default, Relay Connection for GraphQL), idempotency, rate limit headers, OpenAPI security schemes, examples, deprecation markers. Lints with Spectral and validates with Schemathesis/Dredd/Prism. Triggers on "create openapi spec", "generate swagger", "write a proto file", "extract openapi from code", "graphql schema", "document my api", "spec my endpoints". Produces a complete, lintable, CI-gateable spec.
argument-hint: '<format — openapi | graphql | protobuf> [scope — code path OR domain description]'
allowed-tools: Agent, Read, Grep, Glob, Bash, Edit, Write, TodoWrite, WebSearch, WebFetch, mcp__goodmem__goodmem_memories_retrieve, mcp__goodmem__goodmem_memories_get, mcp__context7__resolve-library-id, mcp__context7__query-docs, mcp__plugin_serena_serena__activate_project, mcp__plugin_serena_serena__get_symbols_overview, mcp__plugin_serena_serena__find_symbol, mcp__plugin_serena_serena__list_dir, mcp__plugin_serena_serena__search_for_pattern
---

# Create API Spec

Dispatches the api-expert agent to generate or update an API specification.

## Step 1: Determine format + approach

| Format | Use when |
|---|---|
| **OpenAPI 3.1** | REST API, external consumers, SDK gen with Stainless/Speakeasy |
| **GraphQL SDL** | GraphQL API, client-driven multi-service composition |
| **Protobuf (.proto)** | gRPC service, internal high-performance comms, polyglot |

| Approach | Use when |
|---|---|
| **Code-first (extract)** | Existing codebase with framework auto-gen (FastAPI, Hono, NestJS, Fastify with @fastify/swagger) |
| **Contract-first (write)** | New service, API design in progress, external consumer requirements |
| **Hybrid** | Existing code but spec drifted — extract + refine manually |

## Step 2: Dispatch api-expert

```
Agent({
  description: "API spec: <format>",
  subagent_type: "api-expert:api-expert",
  // model omitted — inherits the session model (always the strongest available Claude)
  prompt: "<see briefing below>"
})
```

### Briefing

```
ORIGINAL USER REQUEST: <verbatim>

WORKFLOW: spec-generation

FORMAT: <openapi / graphql / protobuf>
APPROACH: <code-first / contract-first / hybrid>
SCOPE: <code path OR domain description>
DESTINATION FILE: <where to write — default: openapi.yaml / schema.graphql / service.proto at repo root>

Read ${CLAUDE_PLUGIN_ROOT}/references/schema-design.md FIRST, then client-access.md (for SDK gen implications).

DELIVERABLES:

OpenAPI 3.1:
1. info section with title, description, version (SemVer), license, contact
2. servers list with prod/staging/dev URLs
3. security schemes:
   - OAuth 2.1 with PKCE flows
   - JWT bearer
   - API key (if applicable)
4. paths — one per endpoint with:
   - operationId (camelCase, unique)
   - summary + description (clear + concise)
   - parameters (path, query, header) with types, examples, descriptions
   - requestBody with content-types and schemas
   - responses for 200/201/400/401/403/404/409/422/429/500 AT MINIMUM
   - RFC 9457 Problem JSON for all error responses
5. components/schemas with:
   - Resource models (User, Order, Invoice, etc.)
   - Error model (extends Problem)
   - Pagination envelope (cursor-based)
   - Reusable parameters (PaginationCursor, SortParam, FilterParam)
6. Rate limit headers documented via x-rate-limit extensions OR RateLimit-* response headers
7. Deprecation markers on sunset endpoints (`deprecated: true` + `x-sunset` date)
8. Security requirements per endpoint
9. Tag taxonomy (group by resource/workflow, not alphabetical)
10. webhooks section if applicable
11. Linted with Spectral rules (default: Zalando-style)
12. Validated that code matches spec (if hybrid) — run Schemathesis or Prism validation-proxy mentally

GraphQL SDL:
1. Schema with Query + Mutation + Subscription (if needed)
2. Types (object, interface, union, input, enum, scalar) with descriptions
3. Custom scalars: DateTime, UUID, JSONObject
4. Relay Connection spec for paginated fields (edges/node/pageInfo/cursor)
5. @deprecated directive on sunsetting fields with reason
6. Directives: @auth, @rateLimit, @cacheControl
7. Error handling via union types or errors extension object
8. Federation directives if subgraph (@key, @external, @requires)
9. Schema registry integration notes (Apollo GraphOS / Hive)

Protobuf:
1. syntax = "proto3"
2. Package declaration (reverse-DNS namespace)
3. Import "google/protobuf/timestamp.proto" + well-known types as needed
4. Service definitions with rpc methods
5. Message definitions with:
   - Field numbers assigned (never reuse reserved numbers)
   - Reserved fields for removed field numbers
   - optional keyword for nullable fields (proto3 2020+)
   - oneof for polymorphic fields
   - map<K,V> for associative arrays
6. Streaming RPCs clearly marked (unary / server-stream / client-stream / bidi)
7. Common messages (Pagination, Error, Timestamp) in a shared .proto
8. Backward compatibility preserved (never reuse field numbers or tag values)

LINT + VALIDATE:
- OpenAPI: run `redocly lint <file>` or `spectral lint <file>` (install via npx if needed)
- GraphQL: run `graphql-inspector validate` if available
- Protobuf: run `buf lint` + `buf breaking` if available

WRITE THE SPEC FILE at destination.
UPDATE any auto-doc serving code (e.g., route that serves /openapi.json).
ADD CI step to validate spec on push (stretch goal).

CONSTRAINTS:
- Preserve existing operationIds (clients depend on them)
- Preserve existing error model shape (breaking change otherwise)
- Match project naming conventions (camelCase vs snake_case)
- Use project's existing framework's auto-gen if present (don't duplicate)

PROCEED. Read the reference files above, query goodmem for prior spec generation learnings if configured, then produce the spec.
```

## Step 3: Verify, then relay the spec

Before presenting, verify: the spec file exists at the stated destination; the lint step actually
ran with its output captured (a claim of "lints clean" without pasted output does not count); error
responses are present on every operation (OpenAPI) or the equivalent error contract exists
(GraphQL/Protobuf). Missing pieces → re-query the agent before relaying.

Present the written file path + lint results. Offer to:
- Generate SDK clients from the spec (Stainless, Speakeasy, openapi-typescript)
- Add spec to CI (lint + breaking change detection)
- Create contract tests (Pact or Schemathesis) based on the spec

## Execution mode

The dispatched agent inherits the session model — always the strongest available Claude, never a pinned or dated model. If the session model is already the strongest tier and the task is important or complicated, this skill may generate the spec inline in the main context instead of dispatching a separate agent. Never block on, or wait for, a model that isn't the session model.

## Never do

- Reuse Protobuf field numbers (breaking change)
- Omit error schemas from OpenAPI (spec is half-done without them)
- Skip linting — unlinted specs fail CI later
- Change existing operationIds / GraphQL field names without deprecation flow
