---
name: cross-project-api-audit
description: |-
  Run a comprehensive API audit across 3+ repositories in parallel. Dispatches the api-team-lead agent (runs on the session model, always the strongest available Claude), which maps each repo's API surface, partitions work into 4-10 focused scopes (public API, auth, authz, security, performance, observability, inter-service, testing, lifecycle, deployment), dispatches parallel api-expert sub-agents (bounded by the fan-out budget), and consolidates findings into a unified report with overall verdict + per-repo deltas + cross-cutting pattern detection. Takes 20-90 minutes depending on repo count and scope. Triggers ONLY on explicit phrases — "cross-project api audit", "multi-repo api audit", "audit all my apis", OR the `--team` flag on other skills. Do NOT trigger on generic "audit my api" (single-repo → use `audit-api-security` or `review-api`).
argument-hint: '<repo list comma-separated OR "all" to auto-discover> [--scope=<specific scopes>]'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__goodmem__goodmem_memories_retrieve, mcp__goodmem__goodmem_memories_get, mcp__plugin_serena_serena__activate_project, mcp__plugin_serena_serena__list_dir, mcp__plugin_serena_serena__search_for_pattern
---

# Cross-Project API Audit

Dispatches the api-team-lead agent for large multi-repo audits.

## Step 1: Confirm the team-mode trigger

ONLY proceed if the user explicitly asked for cross-project work:
- Phrase contains "cross-project api audit", "multi-repo api audit", or "audit all my apis"
- OR `--team` flag was passed to another skill that then routed here
- OR scope spans 3+ repositories

If not, STOP. Tell the user:
> "This is the team-mode skill for auditing 3+ repos in parallel. For a single repo, use `/api-expert:audit-api-security` or `/api-expert:review-api` instead."

## Step 2: Resolve repo list

| User arg | Resolution |
|---|---|
| `all` | Auto-discover — find all git repositories under your development root containing API-like code (routes/handlers/controllers) |
| Comma-separated list | Resolve each to absolute path |
| Empty | Ask user which repos |

Verify each path is a git repo and has API-like code (skip static sites, doc repos, data repos).

## Step 3: Dispatch api-team-lead

Read this plugin's `agents/api-team-lead.md` (`${CLAUDE_PLUGIN_ROOT}/agents/api-team-lead.md`) and
inline its body (everything below the frontmatter) as the prompt prefix. Do NOT dispatch via
`subagent_type: "api-expert:api-team-lead"` — the `Agent` tool the team lead needs for sub-dispatch
gets silently stripped by plugin-namespaced dispatch (a known Claude Code platform limitation).
Canonical rationale + evidence: the RUNTIME DISPATCH NOTE at the top of the agent file; maintain the
workaround there only.

```
Agent({
  description: "Cross-project API audit",
  subagent_type: "general-purpose",
  // model omitted — inherits the session model (always the strongest available Claude)
  prompt: "<api-team-lead.md body>\n\n<see briefing below>"
})
```

### Briefing

```
ORIGINAL USER REQUEST: <verbatim>

WORKFLOW: cross-project audit

REPOS IN SCOPE:
- <path 1>
- <path 2>
- ...

SCOPE FILTER (optional): <if user specified --scope=auth,security etc.>

PROCEED WITH YOUR STANDARD TEAM LEAD WORKFLOW:

1. Map each repo (language, framework, API style, endpoint count)
2. Partition into 4-10 focused scopes from the canonical scope list
3. Dispatch api-expert sub-agents in parallel — scale to natural breadth within the fan-out budget (≤10/wave, ≤20/turn; beyond that get explicit user sign-off); use sequential waves (sized to orchestrator context, not a hard 4) if approaching the wave cap or session-reset recurs
4. Consolidate findings into unified report
5. If goodmem is configured, write a meta-learning

Each sub-agent should focus on 1-3 reference files relevant to its scope, not all of them.

EXPECTED DURATION: 20-90 minutes depending on repo count.

REPORT BACK to me with:
- FILE: path where you wrote the unified report (default: /tmp/api-audit-<date>.md)
- LINES: line count
- SCOPES AUDITED: <list>
- SUB-AGENTS DISPATCHED: <count>
- OVERALL VERDICT: <1-2 sentences>
- CRITICAL FINDINGS: <count>
- HIGH FINDINGS: <count>
- CROSS-CUTTING PATTERNS: <count of 3+ repo patterns>
- META-LEARNING ID: <goodmem ID written, if configured>
```

## Step 4: Verify, then relay the report

When the team lead returns, first verify every REPORT BACK field is present (FILE through
META-LEARNING ID) and that the report file at FILE actually exists with the stated line count. Any
missing field → query the team lead for it before presenting; never relay a partial report as
complete.

Then:
1. Present the overall verdict + critical findings summary
2. Show cross-cutting patterns (the highest-leverage fixes)
3. Offer to drill into per-repo deltas
4. Offer to create remediation tasks for the top findings

## Step 5: Follow-up

- If user wants to apply fixes, offer to dispatch api-expert per-repo in fix mode
- If the audit revealed systemic issues, suggest a design document to prevent recurrence
- If cross-cutting patterns suggest shared infrastructure, propose a shared middleware package

## Execution mode

The dispatched api-team-lead orchestrator, and every api-expert sub-agent it fans out to, inherit the session model — always the strongest available Claude, never a pinned or dated model. Never block on, or wait for, a model that isn't the session model. Parallel fan-out does not change this — every sub-agent resolves to the same session model.

## Never do

- Dispatch for single-repo work (use audit-api-security or review-api instead)
- Skip the scope partitioning — unfocused parallel audits produce noise
- Cap agent count out of caution while still inside the fan-out budget (≤10/wave, ≤20/turn) — scale to natural breadth within that budget; beyond it, get explicit user sign-off before dispatching more
- Consolidate findings silently across scopes — always show which scopes flagged what
- Emit advice without at least one sub-agent's evidence backing it
