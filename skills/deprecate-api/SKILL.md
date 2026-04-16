---
name: deprecate-api
description: |-
  Execute the API deprecation and sunset workflow per RFC 9745 (Deprecation header) and RFC 8594 (Sunset header). Plans the notice period (12-24 months typical), writes migration guide, adds `Deprecation` + `Sunset` response headers with Link to migration docs, marks `deprecated: true` in OpenAPI spec and `@deprecated` in GraphQL SDL, notifies consumers (email + dashboard + changelog + blog), tracks usage telemetry to identify heavy users, and enforces the sunset with HTTP 410 Gone at the agreed date. Triggers on "deprecate this endpoint", "sunset v1", "retire the old api", "remove a field", "mark as deprecated", "plan a sunset". Produces a complete deprecation rollout.
argument-hint: '<what to deprecate — endpoint, field, version, whole API>'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
---

# Deprecate API

Dispatches the api-expert agent with a deprecation briefing.

## Step 1: Scope + timeline

| Info | Why |
|---|---|
| What's deprecating (endpoint / field / version / whole API) | Scope |
| Replacement (if exists) | Required in migration guide |
| Consumer audience (internal only / external partners / public) | Determines notice period |
| Target sunset date (absolute date, RFC 1123 format) | Must be >= 12 months for external (>= 24 months is GitHub-style best practice) |
| Current usage telemetry available? | Can we identify heavy users to notify directly? |

## Step 2: Dispatch api-expert

```
Agent({
  description: "API deprecate: <what>",
  subagent_type: "api-expert:api-expert",
  model: "opus",
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
1. Deprecation plan with dates (announcement, deprecation active, sunset, hard cutoff)
2. HTTP headers (RFC 9745 Deprecation + RFC 8594 Sunset)
3. OpenAPI spec update (deprecated: true + x-sunset)
4. GraphQL SDL update (@deprecated directive) if applicable
5. Code-level deprecation markers
6. Changelog entry
7. Migration guide document
8. Communication plan
9. Usage telemetry to set up
10. Enforcement after sunset (410 Gone + RFC 9457 body)
11. Internal cleanup plan (after grace period)

PROCEED.
```

## Step 3: Relay the plan

Present the timeline + all code diffs + migration guide draft. User approves + schedules the announcement.

## Never do

- Set Sunset date earlier than Deprecation date (RFC violation)
- Deprecate without a replacement or explicit "being removed, no replacement" notice
- Use passive header-only notification for external partners
- Remove the endpoint on the sunset date — serve 410 Gone for 30-90 days grace
- Forget to update the OpenAPI / GraphQL spec — spec drift is a downstream DX bug
- Skip the telemetry — you need usage data to know when it's safe to actually remove
