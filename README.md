# api-expert

Claude Code plugin for expert API design, review, debugging, optimization, security auditing, spec generation, migration, and deprecation.

Backed by embedded reference files covering architecture patterns, schema design, authentication, authorization, security hardening, client access, performance, observability, testing, inter-service communication, and API lifecycle.

## Installation

```bash
# Clone into your Claude Code plugins directory
cd ~/.claude/plugins
git clone https://github.com/TheMizeGuy/api-expert-public.git api-expert
```

Or add to an existing project:

```bash
cd your-project
mkdir -p .claude/plugins
git clone https://github.com/TheMizeGuy/api-expert-public.git .claude/plugins/api-expert
```

## Skills

| Skill | Slash command | Purpose |
|---|---|---|
| `api-expert` | `/api-expert:api-expert` | Main entry — classifies request and routes to the right workflow |
| `design-api` | `/api-expert:design-api` | Design a new API from scratch |
| `review-api` | `/api-expert:review-api` | Comprehensive API review with severity-tagged findings |
| `debug-api` | `/api-expert:debug-api` | Systematic API bug diagnosis |
| `optimize-api` | `/api-expert:optimize-api` | Performance optimization (measure first, then fix) |
| `audit-api-security` | `/api-expert:audit-api-security` | OWASP API Top 10 2023 security audit |
| `create-api-spec` | `/api-expert:create-api-spec` | Generate OpenAPI/GraphQL/Protobuf specs |
| `migrate-api` | `/api-expert:migrate-api` | Version upgrades, protocol changes, framework migrations |
| `deprecate-api` | `/api-expert:deprecate-api` | RFC 9745/8594 deprecation and sunset workflow |
| `cross-project-api-audit` | `/api-expert:cross-project-api-audit` | Multi-repo audit (3+ repos, team mode) |

## Agents

| Agent | Model | Purpose |
|---|---|---|
| `api-expert` | Opus | Single-repo API expert — design, review, debug, optimize, secure, spec, migrate, deprecate |
| `api-team-lead` | Opus | Multi-repo orchestrator — partitions work, dispatches parallel api-expert sub-agents, consolidates findings |

## Usage

### Natural language

```
> My GraphQL API is returning intermittent 500s under load
```

The plugin classifies the request, dispatches the api-expert agent with the right workflow, and returns a structured report.

### Slash commands

```
/api-expert:design-api Multi-tenant task management API with iOS + web clients
/api-expert:review-api                    # reviews uncommitted changes
/api-expert:review-api all                # reviews whole project
/api-expert:review-api src/api/orders.ts  # single file
/api-expert:debug-api Webhook delivery failing silently
/api-expert:optimize-api GET /users/:id is p95 1.2s, target 200ms
/api-expert:audit-api-security            # full OWASP walk
/api-expert:create-api-spec openapi src/routes/
/api-expert:migrate-api REST v1 to REST v2 with evolutionary versioning
/api-expert:deprecate-api GET /v1/users/{id}/profile
/api-expert:cross-project-api-audit api,web,mobile,admin,worker
```

### Agent dispatch

```
Agent({ subagent_type: "api-expert:api-expert", model: "opus", prompt: "..." })
Agent({ subagent_type: "api-expert:api-team-lead", model: "opus", prompt: "..." })
```

## Reference files

The `references/` directory contains distilled knowledge on:

| File | Domain |
|---|---|
| `architecture-patterns.md` | Monolith/microservices/BFF, REST/GraphQL/gRPC/tRPC/Connect |
| `schema-design.md` | OpenAPI 3.1, Protobuf, GraphQL SDL, versioning, RFC 9457 errors |
| `authentication.md` | OAuth 2.1, JWT, sessions, BFF, WebAuthn, SAML/SCIM |
| `authorization.md` | RBAC/ABAC/ReBAC, OPA/Cerbos/SpiceDB/Cedar/CASL, PostgreSQL RLS |
| `security-hardening.md` | OWASP API Top 10 2023, TLS 1.3, CORS, supply chain |
| `client-access.md` | SDK generation (Stainless/Speakeasy/Fern), iOS/Android/Web/CLI |
| `performance-caching.md` | Rate limiting, Redis, HTTP caching, CDN, connection pooling |
| `observability.md` | Structured logging, OpenTelemetry, Prometheus, SLOs |
| `testing.md` | Pact, Schemathesis, k6, ZAP/Semgrep, mutation testing |
| `inter-service.md` | Service meshes, circuit breakers, Kafka/RabbitMQ/NATS, sagas |
| `documentation-lifecycle.md` | Docs platforms, versioning, RFC 9745+8594 deprecation |

## Dependencies

| Dependency | Required | Purpose |
|---|---|---|
| context7 MCP | Optional | Library documentation lookup |

All other tools used are Claude Code built-ins (Read, Grep, Glob, Bash, Edit, Write, WebSearch, WebFetch, TodoWrite).

## Protocol coverage

- REST / OpenAPI 3.1
- GraphQL / Apollo Federation
- gRPC / Protocol Buffers / Connect-RPC
- tRPC
- WebSocket / SSE
- Webhooks

## License

MIT
