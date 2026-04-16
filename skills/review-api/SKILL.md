---
name: review-api
description: |-
  Comprehensive API review — endpoints, spec (OpenAPI/GraphQL/Protobuf), code, architecture, and docs. Checks for OWASP API Top 10 2023 violations, authentication/authorization gaps, missing idempotency, pagination correctness, error format (RFC 9457), rate limiting, observability hooks, contract drift, and codebase convention adherence. Triggers on "review my api", "audit my api", "check my endpoint", "api code review", "review this openapi spec", "is my api design sound", "feedback on my api". Produces severity-tagged findings (CRITICAL/HIGH/MEDIUM/LOW/NIT) with concrete fixes and evidence.
argument-hint: '[optional: scope — diff / staged / all / file path / URL]'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
---

# Review API

Dispatches the api-expert agent with a review-workflow briefing.

## Step 1: Resolve scope

| User arg | Scope |
|---|---|
| (empty) or `diff` | Uncommitted + staged changes (default) |
| `staged` | Only staged changes |
| `pr` | Diff vs main/master |
| `<file>` | Single file |
| `<directory>` | All source files in dir |
| `all` | Entire project API surface |
| URL to OpenAPI spec | Fetch + review the spec |

Resolve to absolute paths. Run `git status` to confirm working tree state if scope is diff-based.

## Step 2: Dispatch api-expert

```
Agent({
  description: "API review: <scope>",
  subagent_type: "api-expert:api-expert",
  model: "opus",
  prompt: "<see briefing below>"
})
```

### Briefing

```
ORIGINAL USER REQUEST: <verbatim>

WORKFLOW: review

SCOPE: <resolved scope — files, diff range, or spec URL>

WORKING DIRECTORY: <absolute path>

Read relevant reference files from ${CLAUDE_PLUGIN_ROOT}/references/ based on what you find in the codebase.

DELIVERABLES:
1. File inventory (what you read)
2. Severity-tagged findings table (CRITICAL/HIGH/MEDIUM/LOW/NIT)
3. Category coverage checklist (OWASP, auth, authz, errors, idempotency, pagination, rate limiting, caching, CORS, input validation, logging security, observability, contracts, versioning, conventions)
4. Top 3 priorities (impact x effort)
5. Verification steps the user can run after applying fixes
6. References (reference files, RFCs, URLs)

CONSTRAINTS:
- For CRITICAL/HIGH findings, provide runnable code fixes
- Grep call sites for any shared utility you'd modify
- Preserve behavioral contracts
- If the codebase uses a specific auth library or ORM, stay within it

PROCEED.
```

## Step 3: Relay findings

Present the Summary + severity-tagged table. Offer to apply fixes one-by-one starting with CRITICAL.

## Never do

- Suggest fixes that break existing callers without highlighting the call sites
- Collapse severity — if something is CRITICAL, say so; don't soften to HIGH
- Add emojis
