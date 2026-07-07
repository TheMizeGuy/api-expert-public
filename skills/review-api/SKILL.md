---
name: review-api
description: |-
  Full-surface API review — endpoints, spec (OpenAPI/GraphQL/Protobuf), code, architecture, and docs. Checks for OWASP API Top 10 2023 violations, authentication/authorization gaps, missing idempotency, pagination correctness, error format (RFC 9457), rate limiting, observability hooks, contract drift, and codebase convention adherence. Triggers on "review my api", "audit my api", "check my endpoint", "api code review", "review this openapi spec", "is my api design sound", "feedback on my api". Produces severity-tagged findings (CRITICAL/HIGH/MEDIUM/LOW/NIT) with concrete fixes and cited evidence.
argument-hint: '[optional: scope — diff / staged / all / file path / URL]'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__goodmem__goodmem_memories_retrieve, mcp__goodmem__goodmem_memories_get, mcp__context7__resolve-library-id, mcp__context7__query-docs, mcp__plugin_serena_serena__activate_project, mcp__plugin_serena_serena__get_symbols_overview, mcp__plugin_serena_serena__find_symbol, mcp__plugin_serena_serena__find_referencing_symbols, mcp__plugin_serena_serena__list_dir, mcp__plugin_serena_serena__search_for_pattern
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
If the scope is diff-based and the tree is clean (nothing uncommitted or staged), STOP and ask the
user to name a scope (`all` / `<file>` / `pr`) instead of dispatching a review against an empty diff.

## Step 2: Dispatch api-expert

Resolve the shared OWASP checklist to an absolute path first:
`${CLAUDE_PLUGIN_ROOT}/skills/audit-api-security/references/owasp-api-top-10-2023.md`. Inline it as
OWASP CHECKLIST FILE in the briefing.

```
Agent({
  description: "API review: <scope>",
  subagent_type: "api-expert:api-expert",
  // model omitted — inherits the session model (always the strongest available Claude)
  prompt: "<see briefing below>"
})
```

### Briefing

```
ORIGINAL USER REQUEST: <verbatim>

WORKFLOW: review

SCOPE: <resolved scope — files, diff range, or spec URL>

WORKING DIRECTORY: <absolute path>

OWASP CHECKLIST FILE: <absolute path — inlined by the dispatcher>

Read relevant reference files from ${CLAUDE_PLUGIN_ROOT}/references/ based on what you find in the codebase.

REVIEW METHOD (follow this order):
1. Inventory the surface — list every endpoint/operation in scope with method, path, handler location
2. Read cross-cutting middleware FIRST (auth, error handler, rate limiting, logging) — most findings
   concentrate there, and knowing what the middleware already guarantees prevents false positives
   on individual endpoints
3. Walk each endpoint's data path in order: input validation → authorization predicate →
   query/persistence → response shaping
4. Walk the category coverage list below; for the OWASP row, read the OWASP CHECKLIST FILE and walk
   its items — never mark that box from memory
5. Only then write findings — locate evidence first, assign severity second; a finding without
   file:line evidence is not a finding

DELIVERABLES:
1. File inventory (what you read)
2. Severity-tagged findings table:

   | Severity | Location | Category | Finding | Evidence | Fix |
   |---|---|---|---|---|---|
   | CRITICAL | src/auth.ts:42 | Auth | Missing aud validation on JWT | authentication.md SS JWT | <code block> |
   | HIGH | ... | ... | ... | ... | ... |
   | MEDIUM | ... | ... | ... | ... | ... |
   | LOW | ... | ... | ... | ... | ... |
   | NIT | ... | ... | ... | ... | ... |

3. Category coverage (explicitly note what was checked):
   - [✓/✗] OWASP API Top 10 2023 (BOLA, Broken Auth, Broken Object Property Auth, Unrestricted Resource Consumption, BFLA, Unrestricted Access to Sensitive Business Flows, SSRF, Security Misconfiguration, Improper Inventory Management, Unsafe Consumption of APIs)
   - [✓/✗] Authentication correctness (OAuth 2.1 PKCE, JWT validation, session security)
   - [✓/✗] Authorization correctness (tenant isolation, BOLA guards, row-level security)
   - [✓/✗] Error format (RFC 9457 Problem JSON, consistent shape, no stack traces leaked)
   - [✓/✗] Idempotency (Idempotency-Key for POST, replay semantics)
   - [✓/✗] Pagination (cursor-based for scale, Relay Connection for GraphQL)
   - [✓/✗] Rate limiting (per-IP + per-user + per-tenant, 429 + Retry-After + X-RateLimit-*)
   - [✓/✗] HTTP caching (Cache-Control, ETag, Vary — if public API)
   - [✓/✗] CORS (allowlist, no wildcard + credentials)
   - [✓/✗] Input validation (Zod/Pydantic/etc. at boundaries only)
   - [✓/✗] Logging security (no secrets, no PII, redaction configured)
   - [✓/✗] Observability (OTel instrumentation, correlation IDs, structured logs)
   - [✓/✗] Contract correctness (OpenAPI/GraphQL/Protobuf matches implementation)
   - [✓/✗] Version contract (SemVer for SDK, deprecation headers for endpoints)
   - [✓/✗] Project convention adherence (naming, error shape, logging lib, pkg manager)

4. Top 3 priorities (impact × effort)
5. Verification steps the user can run after applying fixes
6. References (reference files, RFCs, URLs)

CONSTRAINTS:
- For CRITICAL/HIGH findings, provide runnable code fixes, not just descriptions
- Grep call sites for any shared utility you'd modify
- Preserve behavioral contracts — never suggest changing error strings, status codes, log field names, or metric names without explicit rationale
- If the codebase uses a specific auth library or ORM, stay within it (extend, don't replace)

SEVERITY DEFINITIONS:
- CRITICAL: security or correctness bug; ship-blocker
- HIGH: performance or maintainability risk with user impact
- MEDIUM: pattern deviation or technical debt
- LOW: style or clarity
- NIT: preference / taste

ACCEPTANCE CRITERIA — the review is incomplete unless ALL of these hold:
- All 15 category-coverage rows explicitly marked [✓] or [✗] (an unmarked row = unchecked, redo it)
- Grounded either way: if findings exist, at least one cites a reference file; a clean review (zero findings) is valid and instead cites the reference files consulted in its category-coverage notes
- Every CRITICAL/HIGH finding carries a runnable code fix, not a description
- Any fix touching a shared utility lists its call sites

Proceed with your standard workflow (reference files first, then goodmem if configured, then serena/grep of the codebase, then produce the review).
```

## Step 3: Verify, then relay

Before presenting, check the returned review against the acceptance criteria: 15 coverage rows all
marked, reference grounding present (a reference-file-cited finding when findings exist;
consulted-file citations in the coverage notes when the review is clean), runnable fixes on every
CRITICAL/HIGH, call sites listed for shared-utility fixes. If any check fails, re-query the agent
for the gap — do not present a partial review as complete.

Present the Summary + severity-tagged table. Offer to apply fixes one-by-one starting with CRITICAL, or to dispatch a fresh api-expert agent in fix mode to apply a batch of approved findings.

### Fix mode (when the user approves findings)

Dispatch a fresh api-expert agent whose briefing contains: the approved findings VERBATIM (severity, location, evidence, fix block), the review briefing's constraint set (behavioral contracts, call-site propagation), and this mandate: FIRST run the test/lint suite to capture a baseline and record whether the working tree was already dirty (`git status`); then apply ONLY the approved findings, re-run tests/lint, and report per finding — applied (with diff) or skipped (with reason) — plus baseline vs post-apply output, attributing any failure that already existed at baseline to pre-existing state rather than the fixes. Never fold unapproved findings into the same dispatch.

## Execution mode

The dispatched agent inherits the session model — always the strongest available Claude, never a pinned or dated model. If the session model is already the strongest tier and the task is important or complicated, this skill may run the review workflow inline in the main context instead of dispatching a separate agent. Never block on, or wait for, a model that isn't the session model.

## Never do

- Suggest fixes that break existing callers without highlighting the call sites
- Collapse severity — if something is CRITICAL, say so; don't soften to HIGH
- Relay a review with no reference-file citations anywhere — findings cite reference files when they exist; a clean review cites the files it consulted in the coverage notes
- Add emojis
