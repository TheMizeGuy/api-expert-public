---
name: debug-api
description: |-
  Systematically diagnose API bugs — latency spikes, 5xx errors, timeouts, intermittent failures, connection pool exhaustion, N+1 queries, cascading failures, retry storms, rate limit false positives, auth drift, CORS issues, webhook delivery failures, and cross-service correctness bugs. Triggers on "my api is slow", "intermittent 500s", "timeout cascade", "n+1 queries", "api returning wrong data", "webhook not firing", "cors error", "429 errors when there shouldn't be", "my service keeps crashing", "debug my api". Uses systematic debugging (reproduce → isolate → hypothesize → test → confirm → fix → verify) — never guess.
argument-hint: '<symptom — error message, behavior, or affected endpoint>'
allowed-tools: Agent, Read, Grep, Glob, Bash, Edit, Write, TodoWrite, WebSearch, WebFetch, mcp__goodmem__goodmem_memories_retrieve, mcp__goodmem__goodmem_memories_get, mcp__goodmem__goodmem_memories_create, mcp__context7__resolve-library-id, mcp__context7__query-docs, mcp__plugin_serena_serena__activate_project, mcp__plugin_serena_serena__get_symbols_overview, mcp__plugin_serena_serena__find_symbol, mcp__plugin_serena_serena__find_referencing_symbols, mcp__plugin_serena_serena__list_dir, mcp__plugin_serena_serena__search_for_pattern
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
  // model omitted — inherits the session model (always the strongest available Claude)
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
- Can you (the agent) reproduce locally? Try.
- If yes, note exact reproduction steps
- If no (prod-only), work from telemetry

Step 2 — Isolate
- Grep for recent commits touching affected code path
- Check if issue is request-scoped (per-user / per-tenant / per-endpoint) or global
- Check if issue correlates with time-of-day, deploy, traffic level

Step 3 — Hypothesize
- Generate 3-5 specific hypotheses backed by reference-file knowledge
- Rank by likelihood × testability
- Common culprits by symptom:
  - Slow requests → N+1 queries, connection pool exhaustion, missing indexes, cold cache, downstream timeout, lock contention
  - Intermittent 5xx → race condition, memory pressure, GC pauses, upstream flakiness, unhandled promise rejection
  - Cascading failures → missing circuit breaker, timeout > retry × timeout, connection pool exhaustion
  - Retry storms → no jitter, no retry budget, cascading retries across layers
  - CORS → preflight not handled, wildcard + credentials, missing headers
  - Auth drift → JWT expiry mismatch, clock skew, kid rotation not picked up
  - Webhook not firing → outbox polling dead, broker full, signature verification failing at consumer

Step 4 — Test hypothesis
- For each top hypothesis, identify the SMALLEST test that confirms or refutes it
- Tools: pg_stat_statements (slow queries), EXPLAIN ANALYZE (query plan), OpenTelemetry traces (service-to-service), metrics dashboards (CPU/mem/connections), logs (errors), reproduction script

WORKED EXAMPLE (imitate this form — hypothesis → smallest test → verdict, every hypothesis closed):
  Symptom: p95 latency 1.8s on GET /orders under load; <100ms for a single request
  H1 (likely, cheap): connection pool exhaustion — TEST: pool metrics during load run; waiting=12 with max=10 → CONFIRMED contributing
  H2 (likely): N+1 in order→items loading — TEST: pg_stat_statements query count per request; 41 queries/request → CONFIRMED root cause
  H3 (possible): missing index — TEST: EXPLAIN ANALYZE the top query; index scan, 2ms → REFUTED
  Fix targets H2 (batch with DataLoader), then re-size pool for H1. H3 closed with evidence, not dropped silently.

Step 5 — Confirm root cause
- DO NOT fix until you have evidence-backed root cause
- If multiple hypotheses could be true, test them — don't fix based on the first plausible one

Step 6 — Fix
- Minimal change to address the root cause
- Preserve error strings, status codes, log field names, metric names (behavioral contracts)
- Grep callers of any modified shared code
- Add/update a regression test

Step 7 — Verify
- Reproduce the original symptom — it should no longer occur
- Run the full test suite
- Check no other endpoints regressed
- Document the fix + root cause

DELIVERABLES:
1. Reproduction (successful or note that it's prod-only)
2. Evidence collected (logs, traces, query plans, stack traces)
3. Hypothesis list with likelihood ranking
4. Test results per hypothesis
5. Confirmed root cause (with [P]/[V] grade)
6. Fix (runnable code change)
7. Verification steps the user runs
8. If goodmem is configured, write a learning if the bug took >5min to diagnose or had a non-obvious root cause

PROCEED. Read the reference files above, query goodmem for prior art if configured, then systematically debug.
```

## Step 3: Verify, then relay

Do not relay a diagnosis whose root cause is graded `[recall]` — either the agent tests it (upgrade
to `[V]`) or the relay explicitly labels the diagnosis PROVISIONAL and names the outstanding test.
Check that every listed hypothesis was closed (confirmed or refuted with evidence), not silently
dropped.

Present the root cause + fix + verification steps. Offer to apply the fix immediately or stage it for review.

## Execution mode

The dispatched agent inherits the session model — always the strongest available Claude, never a pinned or dated model. If the session model is already the strongest tier and the task is important or complicated, this skill may run the debugging workflow inline in the main context instead of dispatching a separate agent. Never block on, or wait for, a model that isn't the session model.

## Never do

- Accept the first plausible hypothesis without testing it
- Fix symptoms without identifying root cause
- Modify behavioral contracts (error strings, status codes, log/metric names) as part of the "fix"
- Suppress errors or add empty catches to make symptoms go away
- Skip writing a goodmem learning when the root cause was non-obvious and goodmem is configured
