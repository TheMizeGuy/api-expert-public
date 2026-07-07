---
name: audit-api-security
description: |-
  Full API security audit against OWASP API Top 10 2023. Checks BOLA (Broken Object Level Authorization — present in ~40% of API attacks), Broken Authentication, Broken Object Property Level Authorization, Unrestricted Resource Consumption, BFLA, Unrestricted Access to Sensitive Business Flows, SSRF, Security Misconfiguration, Improper Inventory Management, Unsafe Consumption of APIs. Also: TLS 1.3 config, CORS hardening, CSP, input validation, secret management, supply chain (SBOM, Sigstore, SLSA), logging security (no secrets/PII), error response hardening (no stack traces, no DB schema leaks), rate limiting at auth endpoints, JWT attack surface (alg:none, RS256→HS256 confusion, kid injection). Triggers on "audit my api security", "owasp audit", "security review", "is my api secure", "pentest my api", "will my api pass soc 2", "check for vulnerabilities". Produces a finding-by-finding audit with CRITICAL/HIGH/MEDIUM/LOW severity and concrete remediation code.
argument-hint: '[optional: scope — file, directory, or entire project]'
allowed-tools: Agent, Read, Grep, Glob, Bash, Edit, Write, TodoWrite, WebSearch, WebFetch, mcp__goodmem__goodmem_memories_retrieve, mcp__goodmem__goodmem_memories_get, mcp__goodmem__goodmem_memories_create, mcp__context7__resolve-library-id, mcp__context7__query-docs, mcp__plugin_serena_serena__activate_project, mcp__plugin_serena_serena__get_symbols_overview, mcp__plugin_serena_serena__find_symbol, mcp__plugin_serena_serena__find_referencing_symbols, mcp__plugin_serena_serena__list_dir, mcp__plugin_serena_serena__search_for_pattern
---

# Audit API Security

Dispatches the api-expert agent with a security-audit briefing. Walks OWASP API Top 10 2023 item-by-item with evidence.

## Step 1: Scope + context

| Info | Why |
|---|---|
| Audit scope (whole project / specific file / URL) | Focus |
| Compliance target (SOC 2, HIPAA, PCI-DSS, GDPR, none) | Adjusts severity thresholds |
| Production or pre-prod? | Urgency |
| Auth model in use (OAuth/OIDC/session/API keys) | Attack surface |
| Tenant model (single / multi) | BOLA + cross-tenant risk |

## Step 2: Dispatch api-expert

Resolve the canonical checklist to an absolute path first:
`${CLAUDE_PLUGIN_ROOT}/skills/audit-api-security/references/owasp-api-top-10-2023.md`. Inline that
path as CHECKLIST FILE in the briefing — the dispatched agent reads it directly.

```
Agent({
  description: "API security audit: <scope>",
  subagent_type: "api-expert:api-expert",
  // model omitted — inherits the session model (always the strongest available Claude)
  prompt: "<see briefing below>"
})
```

### Briefing

```
ORIGINAL USER REQUEST: <verbatim>

WORKFLOW: security audit

SCOPE: <resolved scope>
COMPLIANCE TARGET: <SOC 2 / HIPAA / PCI / GDPR / none>
ENVIRONMENT: <prod / pre-prod>
AUTH MODEL: <if known>
TENANT MODEL: <single / multi>

CHECKLIST FILE: <absolute path to references/owasp-api-top-10-2023.md — inlined by the dispatcher>

Read the CHECKLIST FILE first. It is the canonical checklist: all ten OWASP API Top 10 2023 items
with per-item checks, five additional check groups (secret management, JWT attacks, dependency
supply chain, logging security, input validation), the severity decision test, and a worked finding
example to imitate. If the file cannot be read, STOP and report that — do NOT reconstruct the
checklist from memory. Also read ${CLAUDE_PLUGIN_ROOT}/references/security-hardening.md,
authentication.md, and authorization.md for deeper context on specific findings.

WALK EVERY CHECKLIST ITEM. For each, document:
- Status (Pass / Fail / Partial / N/A)
- Evidence (file:line references or absence of controls at a named location)
- Severity if Fail or Partial — apply the severity decision test from the checklist file, cite the rule number
- Remediation code block

DELIVERABLES:
1. Audit table — one row per checklist item (10 OWASP items + 5 additional check groups = 15 rows)
2. All CRITICAL + HIGH findings with file:line + fix code
3. Top 5 priorities by risk × effort
4. Compliance mapping (if target specified): show which findings block which compliance requirement
5. Security tooling recommendations (what's missing: ZAP in CI, Semgrep rules, etc.)
6. If goodmem is configured, write a learning on any non-obvious vulnerability pattern found

ACCEPTANCE CRITERIA — the audit is incomplete unless ALL of these hold:
- Audit table has exactly 15 rows; no item skipped
- Every Fail/Partial row has severity (with decision-test rule number), evidence, and a remediation code block
- Every N/A row states its reason (e.g., "no GraphQL surface")
- Every CRITICAL/HIGH finding cites file:line or names the exact missing control and where it belongs
- No severity softened because the environment is pre-prod
- No secret value reproduced anywhere in the report — cite file:line + a redacted fingerprint only (the checklist's redaction rule)

PROCEED. Read the CHECKLIST FILE and the supporting reference files, query goodmem for prior art on similar audits if configured, then walk the checklist.
```

## Step 3: Verify, then relay

Before presenting, verify the returned audit against the acceptance criteria:
- Count the audit-table rows — 15 (10 OWASP + 5 additional groups); a missing row means an unaudited item
- Every Fail/Partial has severity + evidence + remediation code
- Every N/A has a stated reason

If any check fails, re-query the agent for the missing rows — do not present a partial audit as complete.

Present the full audit table + CRITICAL/HIGH findings in priority order. Offer to apply fixes one-by-one starting with CRITICAL.

## Execution mode

The dispatched agent inherits the session model — always the strongest available Claude, never a pinned or dated model. If the session model is already the strongest tier and the task is important or complicated, this skill may run the audit inline in the main context instead of dispatching a separate agent. Never block on, or wait for, a model that isn't the session model.

## Never do

- Skip any OWASP Top 10 item — mark N/A with reason, don't omit
- Soften severity — CRITICAL findings are CRITICAL even if the codebase is pre-prod
- Audit without grepping the codebase — OWASP item-by-item walk requires evidence
- Add emojis
- Produce generic advice — every finding cites a specific file:line or "absence of X at Y"
