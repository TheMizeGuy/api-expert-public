---
name: cross-project-api-audit
description: |-
  Run a comprehensive API audit across 3+ repositories in parallel. Dispatches the api-team-lead Opus 4.7 agent, which maps each repo's API surface, partitions work into 4-10 focused scopes (public API, auth, authz, security, performance, observability, inter-service, testing, lifecycle, deployment), dispatches parallel api-expert sub-agents (max 4 concurrent per Max plan), and consolidates findings into a unified report with overall verdict + per-repo deltas + cross-cutting pattern detection. Takes 20-90 minutes depending on repo count and scope. Triggers ONLY on explicit phrases — "cross-project api audit", "multi-repo api audit", "audit all my apis", OR the `--team` flag on other skills. Do NOT trigger on generic "audit my api" (single-repo → use `audit-api-security` or `review-api`).
argument-hint: '<repo list comma-separated OR "all" to auto-discover> [--scope=<specific scopes>]'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
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
| `all` | Auto-discover — find all git repos containing API-like code (routes/handlers/controllers) |
| Comma-separated list | Resolve each to absolute path |
| Empty | Ask user which repos |

Verify each path is a git repo and has API-like code (skip static sites, doc repos, data repos).

## Step 3: Dispatch api-team-lead

```
Agent({
  description: "Cross-project API audit",
  subagent_type: "api-expert:api-team-lead",
  model: "opus",
  prompt: "<see briefing below>"
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
3. Dispatch api-expert sub-agents sequentially in waves of 4 (rate limit)
4. Consolidate findings into unified report

Read reference files from ${CLAUDE_PLUGIN_ROOT}/references/ for grounding. Each sub-agent should focus on 1-3 reference files relevant to its scope, not all of them.

REPORT BACK with:
- FILE: path where you wrote the unified report (default: /tmp/api-audit-<date>.md)
- SCOPES AUDITED: <list>
- SUB-AGENTS DISPATCHED: <count>
- OVERALL VERDICT: <1-2 sentences>
- CRITICAL FINDINGS: <count>
- HIGH FINDINGS: <count>
- CROSS-CUTTING PATTERNS: <count of 3+ repo patterns>
```

## Step 4: Relay the report

When the team lead returns:
1. Present the overall verdict + critical findings summary
2. Show cross-cutting patterns (the highest-leverage fixes)
3. Offer to drill into per-repo deltas
4. Offer to create remediation tasks for the top findings

## Never do

- Dispatch for single-repo work (use audit-api-security or review-api instead)
- Skip the scope partitioning — unfocused parallel audits produce noise
- Dispatch more than 4 concurrent sub-agents (rate limit)
- Consolidate findings silently across scopes — always show which scopes flagged what
