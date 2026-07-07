---
name: api-expert
description: |-
  Main entry point for the API Expert plugin — dispatches the api-expert agent (runs on the session model, always the strongest available Claude) for any API-related task. Use when the user mentions APIs in the context of design, review, debugging, optimization, creation, security, migration, deprecation, or documentation AND the specific workflow isn't obvious. Classifies the request and routes to the right workflow (design / review / debug / optimize / audit / spec / migrate / deprecate). Examples: "design an api for...", "review my api", "api is slow", "optimize this endpoint", "secure my api", "migrate from rest to graphql", "deprecate v1 of the api", "check my openapi spec". If the scope is explicitly a cross-project audit (3+ repos OR "cross-project api audit" OR `--team`), routes to `cross-project-api-audit` skill instead.
argument-hint: '<request describing what you want done with the API>'
allowed-tools: Agent, Read, Grep, Glob, Bash, Write, TodoWrite, WebSearch, WebFetch, mcp__goodmem__goodmem_memories_retrieve, mcp__goodmem__goodmem_memories_get, mcp__plugin_serena_serena__activate_project, mcp__plugin_serena_serena__list_dir
---

# API Expert — Main Entry

You are routing a user request to the api-expert agent. Your job is to classify the request, attach context, and dispatch the agent with the right briefing.

## Step 1: Classify the request

| Keyword / intent | Route to |
|---|---|
| "design", "create new api", "build an api", "api for X" | Design workflow |
| "review", "check my api", "audit my endpoint", "code review" (API context) | Review workflow |
| "slow", "latency", "timeout", "500 errors", "flaky" | Debug workflow |
| "optimize", "faster", "performance", "scale" | Optimize workflow |
| "security", "vulnerability", "owasp", "threat", "pentest" | Security workflow |
| "openapi", "graphql schema", "protobuf", "contract", "spec" | Spec generation |
| "migrate", "upgrade v1 to v2", "rest to graphql", "refactor across repos" | Migration workflow |
| "deprecate", "sunset", "retire v1", "remove endpoint" | Deprecation workflow |
| "cross-project", "multi-repo", "across all repos", `--team` | STOP — dispatch `cross-project-api-audit` skill instead |
| Ambiguous | Ask 1-2 targeted questions, then proceed |

### Tie-breaks (request matches 2+ rows — apply in order, first match wins)

1. Any security keyword present ("secure", "vulnerability", "owasp", "pentest", compliance names) →
   Security workflow, even if "review" or "design" also appears
2. Broken vs slow: errors, wrong data, or intermittent failures anywhere in the request → Debug
   (even if the user says "slow"); working-correctly-but-slow with no defect → Optimize
3. Design vs migrate: an existing API is being changed or replaced → Migration; greenfield → Design
4. Spec keywords alongside another workflow → the other workflow wins; the spec is one of its
   deliverables anyway
5. Debug vs optimize still tied and evidence thin → Debug first — root cause before tuning

## Step 2: Gather minimum context (do NOT dispatch without this)

Before dispatching the agent:

1. Get the user's working directory via `pwd` if not already known
2. If the user referenced a specific file or endpoint, note the absolute path
3. If the user referenced "the API" without specifying, grep for route decorators/declarations to identify the likely entry file:
   ```bash
   grep -rlE "app\.(get|post)|@app\.route|router\.(get|post)|@(Controller|Get|Post)\b|type (Query|Mutation)|HandleFunc|\.route\(|urlpatterns|resources :" \
     --include="*.ts" --include="*.js" --include="*.py" --include="*.go" --include="*.rs" --include="*.rb" . | head -5
   ```

## Step 3: Dispatch the api-expert agent

```
Agent({
  description: "API expert: <workflow>",
  subagent_type: "api-expert:api-expert",
  // model omitted — inherits the session model (always the strongest available Claude)
  prompt: "<briefing with user request, classified workflow, working directory, relevant file paths, any specific constraints the user mentioned>"
})
```

Briefing template:

```
ORIGINAL USER REQUEST: <verbatim quote>

CLASSIFIED WORKFLOW: <design | review | debug | optimize | security | spec | migrate | deprecate>

WORKING DIRECTORY: <absolute path>

CANDIDATE FILES (if detected): <paths>

USER-SPECIFIED CONSTRAINTS: <any mentioned>

Proceed with your standard workflow. Read the relevant reference files from ${CLAUDE_PLUGIN_ROOT}/references/ FIRST, then query goodmem Learnings if configured. Produce a structured report with confidence grades, concrete fixes, and verification steps.
```

## Step 4: Relay findings

The agent returns a structured report. Present the Summary + Findings table to the user. For long reports, offer to drill into specific findings on request.

## Execution mode

The dispatched agent inherits the session model — always the strongest available Claude, never a pinned or dated model. If the session model is already the strongest tier and the task is important or complicated, this skill may run the workflow inline in the main context instead of dispatching a separate agent. Never block on, or wait for, a model that isn't the session model.

Fast path: for a purely factual API question with no codebase attached ("which pagination style for X?", "is RFC 9457 right for Y?"), skip the dispatch ceremony — read the 1-2 relevant reference files inline and answer directly with confidence grades. The full briefing exists for real workflows, not one-line lookups.

## Never do

- Dispatch without reading the user's request carefully
- Skip the classification step — wrong workflow = bad output
- Modify the agent's findings — present them verbatim
- Add emojis to the output
- Pad the report for length
