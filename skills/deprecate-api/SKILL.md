---
name: deprecate-api
description: |-
  Execute the API deprecation and sunset workflow per RFC 9745 (Deprecation header) and RFC 8594 (Sunset header). Plans the notice period (12-24 months typical), writes migration guide, adds `Deprecation` + `Sunset` response headers with Link to migration docs, marks `deprecated: true` in OpenAPI spec and `@deprecated` in GraphQL SDL, notifies consumers (email + dashboard + changelog + blog), tracks usage telemetry to identify heavy users, and enforces the sunset with HTTP 410 Gone at the agreed date. Triggers on "deprecate this endpoint", "sunset v1", "retire the old api", "remove a field", "mark as deprecated", "plan a sunset". Produces a complete deprecation rollout.
argument-hint: '<what to deprecate — endpoint, field, version, whole API>'
allowed-tools: Agent, Read, Grep, Glob, Bash, Edit, Write, TodoWrite, WebSearch, WebFetch, mcp__goodmem__goodmem_memories_retrieve, mcp__goodmem__goodmem_memories_get, mcp__goodmem__goodmem_memories_create, mcp__context7__resolve-library-id, mcp__context7__query-docs, mcp__plugin_serena_serena__activate_project, mcp__plugin_serena_serena__get_symbols_overview, mcp__plugin_serena_serena__find_symbol, mcp__plugin_serena_serena__find_referencing_symbols, mcp__plugin_serena_serena__list_dir, mcp__plugin_serena_serena__search_for_pattern
---

# Deprecate API

Dispatches the api-expert agent with a deprecation briefing.

## Step 1: Scope + timeline

| Info | Why |
|---|---|
| What's deprecating (endpoint / field / version / whole API) | Scope |
| Replacement (if exists) | Required in migration guide |
| Consumer audience (internal only / external partners / public) | Determines notice period |
| Target sunset date (absolute date — ISO in the plan; it renders as an HTTP-date in the `Sunset` header) | Must be ≥ 12 months for external (≥ 24 months is GitHub-style best practice) |
| Current usage telemetry available? | Can we identify heavy users to notify directly? |

## Step 2: Dispatch api-expert

```
Agent({
  description: "API deprecate: <what>",
  subagent_type: "api-expert:api-expert",
  // model omitted — inherits the session model (always the strongest available Claude)
  prompt: "<see briefing below>"
})
```

### Briefing

```
ORIGINAL USER REQUEST: <verbatim>

WORKFLOW: deprecation

WHAT: <endpoint / field / version / whole API>
REPLACEMENT: <path to new or "none — being removed">
AUDIENCE: <internal / external partners / public>
SUNSET DATE (target): <ISO date>
TELEMETRY: <yes / no, what's tracked>

Read ${CLAUDE_PLUGIN_ROOT}/references/documentation-lifecycle.md FIRST.

DELIVERABLES:

1. Deprecation plan with dates:
   - Announcement date (now or +7d)
   - Deprecation active date (`Deprecation` header starts)
   - Sunset date (`Sunset` header value)
   - Migration guide published
   - Hard cutoff date (410 Gone response)
   - Minimum 12 months for external, 6 months for internal-only

2. HTTP headers added:

   RFC 9745 Deprecation (value is Unix timestamp per RFC 9651):
   ```
   Deprecation: @1735689600
   Link: <https://developer.example.com/migrate/v1-to-v2>; rel="deprecation"; type="text/html"
   ```

   RFC 8594 Sunset (value is RFC 1123 date):
   ```
   Sunset: Thu, 31 Dec 2026 23:59:59 GMT
   Link: <https://developer.example.com/sunset/v1>; rel="sunset"; type="text/html"
   ```

   Constraint: Sunset MUST NOT be earlier than Deprecation.

3. OpenAPI spec update:
   - Set `deprecated: true` on the operation
   - Add `x-sunset: "2026-12-31T23:59:59Z"` extension
   - Link to migration guide in `description`

4. GraphQL SDL update (for field deprecation):
   ```graphql
   type User {
     email: String! @deprecated(reason: "Use primaryEmail. Sunset: 2026-12-31.")
     primaryEmail: String!
   }
   ```

5. Code-level deprecation markers:
   - TypeScript: `/** @deprecated Use ... instead. Sunset 2026-12-31. */`
   - Python: `@deprecated("Use ... instead. Sunset 2026-12-31.")` (via `typing_extensions` or `deprecation` library)
   - Go: `// Deprecated: use ... instead. Sunset 2026-12-31.`
   - Swift: `@available(*, deprecated, message: "...")`

6. Changelog entry (Keep a Changelog format):
   ```markdown
   ## [v1.8.0] - 2026-04-14
   ### Deprecated
   - `GET /v1/users/{id}/profile` — use `GET /v2/users/{id}` instead. Sunset 2026-12-31.
   ```

7. Migration guide document — in docs portal or MIGRATION.md:
   - What's changing + why
   - Old vs new request/response side-by-side
   - Code snippets showing migration in each supported language
   - Timeline (deprecation date, sunset date)
   - FAQ
   - How to get help

8. Communication plan:
   - Email all registered API consumers
   - Dashboard banner + persistent warning on deprecated endpoints
   - Blog post (for public API)
   - Forum / Discord / Slack announcement
   - Proactive outreach to top 10 users by volume (telemetry-driven)

9. Usage telemetry to set up:
   - Counter metric: `api_deprecated_endpoint_calls_total{endpoint, consumer_id}`
   - Log each call with `@deprecated: true` attribute
   - Dashboard tracking deprecated endpoint usage trend over time
   - Alert if usage doesn't decrease after month 6

10. Enforcement after sunset:
    - Replace handler with `410 Gone`
    - Response body: RFC 9457 Problem JSON with link to migration guide
    - Preserve the endpoint for 30-90 days after sunset date returning 410 (helps clients that haven't migrated)
    - Then remove handler + 410 response → 404

11. Internal cleanup (after grace period):
    - Remove old code paths
    - Remove old tests
    - Simplify any compatibility middleware

PROCEED. Read the reference file above, query goodmem for prior deprecation learnings if configured, then produce the plan + code changes.
```

## Step 3: Verify, then relay the plan

Before presenting, verify the returned plan: Sunset date is not earlier than the Deprecation date;
the notice period meets the audience minimum (12 months external, 6 months internal); all 11
numbered deliverables are present, including telemetry and the post-sunset 410 grace window.
Missing pieces → re-query the agent before relaying.

Present the timeline + all code diffs + migration guide draft. User approves + schedules the announcement.

## Execution mode

The dispatched agent inherits the session model — always the strongest available Claude, never a pinned or dated model. If the session model is already the strongest tier and the task is important or complicated, this skill may run the deprecation workflow inline in the main context instead of dispatching a separate agent. Never block on, or wait for, a model that isn't the session model.

## Never do

- Set Sunset date earlier than Deprecation date (RFC violation)
- Deprecate without a replacement or explicit "being removed, no replacement" notice
- Use passive header-only notification for external partners (Zalando: require explicit consent)
- Remove the endpoint on the sunset date — serve 410 Gone for 30-90 days grace
- Forget to update the OpenAPI / GraphQL spec — spec drift is a downstream DX bug
- Skip the telemetry — you need usage data to know when it's safe to actually remove
