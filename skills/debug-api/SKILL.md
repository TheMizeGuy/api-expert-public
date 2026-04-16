---
name: debug-api
description: |-
  Systematically diagnose API bugs — latency spikes, 5xx errors, timeouts, intermittent failures, connection pool exhaustion, N+1 queries, cascading failures, retry storms, rate limit false positives, auth drift, CORS issues, webhook delivery failures, and cross-service correctness bugs. Triggers on "my api is slow", "intermittent 500s", "timeout cascade", "n+1 queries", "api returning wrong data", "webhook not firing", "cors error", "429 errors when there shouldn't be", "my service keeps crashing", "debug my api". Uses systematic debugging (reproduce → isolate → hypothesize → test → confirm → fix → verify) — never guess.
argument-hint: '<symptom — error message, behavior, or affected endpoint>'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
---

# Debug API

Dispatches the api-expert agent with a systematic debugging briefing. No guessing — evidence drives every step.

## Step 1: Collect symptoms

Before dispatching, extract or ask:

| Question | Why |
|---|---|
| Exact error message or status code | Filter by severity/type |
| Reproduction steps | Isolate trigger |
| Frequency (every request / 1% / random spikes) | Narrow hypothesis space |
| When it started (deploy correlation?) | Link to recent changes |
| Affected endpoints (one? many?) | Scope of issue |
| Logs / traces available? | Primary evidence |
| Environment (prod / staging / local) | Infra constraints |

Ask only the 1-2 critical questions you can't infer. If user says "just go", proceed with what's given.

## Step 2: Dispatch api-expert

```
Agent({
  description: "API debug: <symptom>",
  subagent_type: "api-expert:api-expert",
  model: "opus",
  prompt: "<see briefing below>"
})
```

### Briefing

```
ORIGINAL USER REQUEST: <verbatim>

WORKFLOW: debug

SYMPTOM: <exact error/behavior>
REPRODUCTION: <steps>
FREQUENCY: <every / N% / random>
ONSET: <when started / deploy correlation>
AFFECTED ENDPOINTS: <list>
LOGS/TRACES: <paths or excerpts if provided>
ENVIRONMENT: <prod/staging/local>

Read ${CLAUDE_PLUGIN_ROOT}/references/performance-caching.md and observability.md, plus any other relevant reference files.

FOLLOW SYSTEMATIC DEBUGGING METHOD:

Step 1 — Reproduce
Step 2 — Isolate (grep recent commits, check if request-scoped or global)
Step 3 — Hypothesize (3-5 specific hypotheses ranked by likelihood x testability)
Step 4 — Test hypothesis (smallest test that confirms or refutes)
Step 5 — Confirm root cause (evidence-backed, do NOT fix until confirmed)
Step 6 — Fix (minimal change, preserve behavioral contracts, grep callers)
Step 7 — Verify (reproduce original symptom — should no longer occur, run full test suite)

DELIVERABLES:
1. Reproduction (successful or note that it's prod-only)
2. Evidence collected (logs, traces, query plans, stack traces)
3. Hypothesis list with likelihood ranking
4. Test results per hypothesis
5. Confirmed root cause (with [P]/[V] grade)
6. Fix (runnable code change)
7. Verification steps the user runs

PROCEED.
```

## Step 3: Relay the diagnosis

Present the root cause + fix + verification steps. Offer to apply the fix immediately or stage it for review.

## Never do

- Accept the first plausible hypothesis without testing it
- Fix symptoms without identifying root cause
- Modify behavioral contracts (error strings, status codes, log/metric names) as part of the "fix"
- Suppress errors or add empty catches to make symptoms go away
