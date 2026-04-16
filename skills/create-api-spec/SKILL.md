---
name: create-api-spec
description: |-
  Generate or update API specifications — OpenAPI 3.1 YAML, GraphQL SDL, or Protocol Buffers (.proto). Supports contract-first (write spec, generate code) and code-first (extract spec from existing code). Adds proper error schemas (RFC 9457 Problem JSON), pagination contracts (cursor-based default, Relay Connection for GraphQL), idempotency, rate limit headers, OpenAPI security schemes, examples, deprecation markers. Lints with Spectral and validates with Schemathesis/Dredd/Prism. Triggers on "create openapi spec", "generate swagger", "write a proto file", "extract openapi from code", "graphql schema", "document my api", "spec my endpoints". Produces a complete, lintable, CI-gateable spec.
argument-hint: '<format — openapi | graphql | protobuf> [scope — code path OR domain description]'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
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
  model: "opus",
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

DELIVERABLES: Complete spec with proper error schemas (RFC 9457), pagination, security schemes, deprecation markers, examples. Lint with Spectral. Validate code matches spec if hybrid.

CONSTRAINTS:
- Preserve existing operationIds (clients depend on them)
- Preserve existing error model shape (breaking change otherwise)
- Match project naming conventions (camelCase vs snake_case)
- Use project's existing framework's auto-gen if present (don't duplicate)

PROCEED.
```

## Step 3: Relay the spec

Present the written file path + lint results. Offer to:
- Generate SDK clients from the spec (Stainless, Speakeasy, openapi-typescript)
- Add spec to CI (lint + breaking change detection)
- Create contract tests (Pact or Schemathesis) based on the spec

## Never do

- Reuse Protobuf field numbers (breaking change)
- Omit error schemas from OpenAPI (spec is half-done without them)
- Skip linting — unlinted specs fail CI later
- Change existing operationIds / GraphQL field names without deprecation flow
