# api-expert

Version 0.2.1

Claude Code plugin for expert API design, review, debugging, optimization, security auditing, spec generation, migration, and deprecation. Runs on the session model — always the strongest available Claude, never a pinned or dated model.

Backed by embedded reference files covering architecture patterns, schema design, authentication, authorization, security hardening, client access, performance, observability, testing, inter-service communication, and API lifecycle, plus a canonical OWASP API Top 10 2023 checklist. Optionally grounds itself further with goodmem (memory) and serena (semantic code navigation) MCP servers when configured — every workflow works standalone from the embedded references if you don't have those.

See [USAGE.md](USAGE.md) for a quickstart, a worked walkthrough per skill, and a troubleshooting table.

## Installation

```bash
# 1. Add this repo as a marketplace
claude plugin marketplace add https://github.com/TheMizeGuy/api-expert-public.git

# 2. Install the plugin
claude plugin install api-expert@api-expert-public

# 3. Restart Claude Code for the plugin to load
```

After restart, verify with `claude plugin list`. Updates ship through the same channel: when a new release lands, run `claude plugin marketplace update api-expert-public` then `claude plugin update api-expert@api-expert-public`, or accept the update prompt in `/plugin`.

Manual alternative: `git clone https://github.com/TheMizeGuy/api-expert-public.git` and load with `claude --plugin-dir <path>`.

## Skills

| Skill | Slash command | Purpose |
|---|---|---|
| `api-expert` | `/api-expert:api-expert` | Main entry — classifies request and routes to the right workflow |
| `design-api` | `/api-expert:design-api` | Design a new API from scratch |
| `review-api` | `/api-expert:review-api` | Full-surface API review with severity-tagged findings |
| `debug-api` | `/api-expert:debug-api` | Systematic API bug diagnosis |
| `optimize-api` | `/api-expert:optimize-api` | Performance optimization (measure first, then fix) |
| `audit-api-security` | `/api-expert:audit-api-security` | OWASP API Top 10 2023 security audit against the canonical 15-row checklist |
| `create-api-spec` | `/api-expert:create-api-spec` | Generate OpenAPI/GraphQL/Protobuf specs |
| `migrate-api` | `/api-expert:migrate-api` | Version upgrades, protocol changes, framework migrations |
| `deprecate-api` | `/api-expert:deprecate-api` | RFC 9745/8594 deprecation and sunset workflow |
| `cross-project-api-audit` | `/api-expert:cross-project-api-audit` | Multi-repo audit (3+ repos, team mode) |

## Agents

| Agent | Purpose |
|---|---|
| `api-expert` | Single-repo API expert — design, review, debug, optimize, secure, spec, migrate, deprecate |
| `api-team-lead` | Multi-repo orchestrator — partitions work, dispatches parallel api-expert sub-agents, consolidates findings |

Both agents run on the session model — the strongest available Claude at dispatch time. Neither agent frontmatter pins a model; there is nothing to configure.

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

### Agent dispatch (advanced)

For single-repo work, dispatch `api-expert` directly:

```
Agent({ subagent_type: "api-expert:api-expert", prompt: "..." })
```

For multi-repo team mode, do NOT dispatch `api-team-lead` by plugin-namespaced `subagent_type` —
plugin-namespaced dispatch silently strips the `Agent` tool the team lead needs to fan out
sub-agents (a Claude Code platform limitation; see the RUNTIME DISPATCH NOTE at the top of
`agents/api-team-lead.md`). Use the `cross-project-api-audit` skill instead — it reads the agent
file's body and dispatches it under `subagent_type: "general-purpose"`, which keeps the `Agent`
tool intact:

```
/api-expert:cross-project-api-audit api,web,mobile,admin,worker
```

In every dispatch, omit `model` — the agent inherits the session model, always the strongest
available Claude at the time it runs.

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

`skills/audit-api-security/references/owasp-api-top-10-2023.md` is a separate, more detailed
checklist: all ten OWASP items with per-item checks, five additional check groups (secrets, JWT
attacks, dependency supply chain, logging security, input validation), a severity decision test, and
a worked finding example. `audit-api-security` and `review-api` both read it directly.

## Dependencies

| Dependency | Required | Purpose |
|---|---|---|
| context7 MCP | Optional | Library documentation lookup |
| goodmem MCP | Optional | Cross-session memory — prior learnings on the same problem, meta-learnings across audits |
| serena MCP | Optional | Semantic code navigation (symbol lookup instead of grep) |

None of these are required. Every skill and agent skips a step gracefully when its MCP server isn't
configured, and works from the embedded reference files plus your codebase alone. If you do use
goodmem, the space IDs referenced in agent/skill bodies are placeholders (`<your-goodmem-...-id>`) —
fill in your own.

All other tools used are Claude Code built-ins (Read, Grep, Glob, Bash, Edit, Write, WebSearch, WebFetch, TodoWrite).

## Protocol coverage

- REST / OpenAPI 3.1
- GraphQL / Apollo Federation
- gRPC / Protocol Buffers / Connect-RPC
- tRPC
- WebSocket / SSE
- Webhooks

## Not for

- Quick "what's a REST API?" questions — just use regular Claude for that
- Implementation-only tasks where the architecture is fixed — use direct coding instead
- Single-line error message lookups — use regular Claude
- Codebases with no API surface at all (a CLI tool, a static site, a data pipeline with no service boundary)

## Related plugins

If you also use these public plugin mirrors from the same author, they complement api-expert:

| Plugin | When to use alongside api-expert |
|---|---|
| `scrum-master-public` | Convert audit findings into kanban stories |
| `railway-operator-public` | Actually deploy the designed API on Railway |
| `typescript-senior-review` | Deeper TypeScript-specific review (complementary) |
| `ios-code-review` (plugin name `ios-senior-review`) | Review the iOS client-side of the API contract |
| `deep-research-public` | Extend the reference files when gaps surface |

## License

MIT
