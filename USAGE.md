# Using api-expert

This is the user guide. It assumes you've already installed the plugin per the README's
Installation section and are working inside a Claude Code session with the plugin active.

## Quickstart

1. **Install** — clone the repo into your plugins directory (README → Installation).
2. **Invoke** — either describe what you need in plain language, or use a slash command directly:
   ```
   > review my API for security issues
   ```
   or
   ```
   /api-expert:audit-api-security
   ```
3. **What happens** — the invoked skill gathers a little context (scope, environment, constraints),
   dispatches the `api-expert` agent with a structured briefing, and the agent reads the relevant
   embedded reference file(s) before producing anything. You get back a structured report:
   Summary → Context → Findings → Code changes → Verification steps → References.
4. **Follow up** — most skills end by offering to apply fixes one at a time, starting with the
   highest-severity finding. You approve each change before it lands.

No configuration is required to get useful output. goodmem, serena, and context7 are optional — if
none are configured, the agent works from the embedded `references/` files and your codebase alone.

## Walkthroughs

### Design a new API (`design-api`)

```
> /api-expert:design-api Multi-tenant invoice API, TypeScript, ~50 RPS, web + mobile clients
```

What happens: the skill asks at most 1-3 clarifying questions if something critical is missing
(here, probably none — the request already answers domain, stack, scale, and clients). It dispatches
`api-expert` with a design briefing. The agent reads `architecture-patterns.md`,
`schema-design.md`, `authentication.md`, and `authorization.md`, then produces:

- An architectural decision record (protocol, monolith vs microservices, auth, authz, DB/caching, deployment — each with a stated rationale)
- A complete OpenAPI 3.1 spec (or GraphQL SDL / Protobuf, depending on protocol choice)
- A runnable scaffold (framework setup, auth middleware, RFC 9457 error handler, rate limiting, pagination, OTel, structured logging)
- A testing plan and a deployment plan
- An observability plan and a lifecycle plan (versioning, deprecation policy)

Output shape: a long structured report. You'll typically be shown the Summary + ADR first, with an
offer to drill into the full spec or scaffold on request.

### Review existing API code (`review-api`)

```
> /api-expert:review-api all
```

What happens: the skill resolves scope (here, the whole project's API surface), reads the canonical
OWASP checklist from `skills/audit-api-security/references/owasp-api-top-10-2023.md`, and dispatches
`api-expert` with a review briefing that walks middleware first, then each endpoint's data path.

Output shape: a severity-tagged findings table (CRITICAL/HIGH/MEDIUM/LOW/NIT) with file:line
evidence and runnable fixes on every CRITICAL/HIGH row, plus a 15-row category coverage checklist
showing exactly what was checked and what wasn't applicable.

### Debug an intermittent failure (`debug-api`)

```
> /api-expert:debug-api GraphQL endpoint returns 500s under load, about 1% of requests
```

What happens: the skill asks only the 1-2 questions it can't infer (e.g., "when did this start?"),
then dispatches `api-expert` with a systematic-debugging briefing: reproduce → isolate →
hypothesize → test → confirm → fix → verify. The agent will not present a fix until a hypothesis is
confirmed with evidence (query plans, pool metrics, traces) — a plausible-but-untested guess gets
flagged as PROVISIONAL, not presented as the answer.

Output shape: root cause (with a confidence grade), a runnable fix, and verification steps you run
yourself to confirm the symptom is gone.

### Security audit (`audit-api-security`)

```
> /api-expert:audit-api-security
```

What happens: the skill resolves scope and compliance target if given, then dispatches `api-expert`
with the full canonical OWASP API Top 10 2023 checklist inlined as an absolute file path. The agent
walks all 15 rows (10 OWASP items + 5 additional check groups: secrets, JWT attacks, dependency
supply chain, logging security, input validation) — none from memory.

Output shape: a 15-row audit table (Status / Evidence / Severity / Remediation), every
CRITICAL/HIGH finding with a runnable fix, and a top-5 priority list by risk × effort.

### Multi-repo audit (`cross-project-api-audit`)

```
> /api-expert:cross-project-api-audit api,web,mobile,admin,worker
```

Only triggers on an explicit cross-project phrase, `--team`, or 3+ named repos — for one repo, use
`review-api` or `audit-api-security` instead. What happens: the skill dispatches the `api-team-lead`
agent body under `subagent_type: "general-purpose"` (never the plugin-namespaced `api-team-lead`
subagent_type — see the RUNTIME DISPATCH NOTE in `agents/api-team-lead.md` for why). The team lead
maps each repo, partitions into 4-10 focused scopes, and fans out `api-expert` sub-agents in
parallel, one per scope.

Output shape: a unified report with an overall verdict, per-repo findings, and cross-cutting
patterns (issues that show up in 3+ repos — fix these once, apply everywhere). Takes 20-90 minutes
depending on repo count.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `/api-expert:*` commands not found | Plugin not cloned into a location Claude Code scans | Re-check the Installation section; confirm the clone landed under `~/.claude/plugins/` or `<project>/.claude/plugins/` |
| Cross-project audit silently does nothing / sub-agents never dispatch | Something invoked `api-team-lead` via `subagent_type: "api-expert:api-team-lead"` directly | Always go through the `cross-project-api-audit` skill, which dispatches under `general-purpose` instead — plugin-namespaced dispatch strips the `Agent` tool the team lead needs |
| Agent output cites goodmem/serena steps it then skips | Those MCP servers aren't configured in your session | Expected — every workflow explicitly frames goodmem/serena queries as "if configured"; install them only if you want cross-session memory or symbol-aware navigation |
| Findings feel generic, no file:line evidence | Scope was too broad or the codebase wasn't grep-able (e.g., a spec-only review with no code) | Narrow scope to a directory or file; for spec-only reviews, evidence will cite spec line numbers instead |
| Security audit table has fewer than 15 rows | The agent skipped an item, or the relay didn't verify before presenting | This violates the skill's acceptance criteria — re-run and ask specifically for the missing checklist rows |
| Deprecation plan proposes a Sunset date before the Deprecation date | Model error against the RFC 9745/8594 constraint | Reject and ask for a corrected timeline — Sunset must never precede Deprecation |
| Want to change severity thresholds for a compliance target | Compliance mapping only runs if you name a target | Re-invoke `audit-api-security` and state the target explicitly (SOC 2, HIPAA, PCI-DSS, GDPR) |
