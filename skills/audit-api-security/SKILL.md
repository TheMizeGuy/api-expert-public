---
name: audit-api-security
description: |-
  Comprehensive API security audit against OWASP API Top 10 2023. Checks BOLA (Broken Object Level Authorization — present in ~40% of API attacks), Broken Authentication, Broken Object Property Level Authorization, Unrestricted Resource Consumption, BFLA, Unrestricted Access to Sensitive Business Flows, SSRF, Security Misconfiguration, Improper Inventory Management, Unsafe Consumption of APIs. Also: TLS 1.3 config, CORS hardening, CSP, input validation, secret management, supply chain (SBOM, Sigstore, SLSA), logging security (no secrets/PII), error response hardening (no stack traces, no DB schema leaks), rate limiting at auth endpoints, JWT attack surface (alg:none, RS256→HS256 confusion, kid injection). Triggers on "audit my api security", "owasp audit", "security review", "is my api secure", "pentest my api", "will my api pass soc 2", "check for vulnerabilities". Produces a finding-by-finding audit with CRITICAL/HIGH/MEDIUM/LOW severity and concrete remediation code.
argument-hint: '[optional: scope — file, directory, or entire project]'
allowed-tools: Agent, Read, Grep, Glob, Bash, TodoWrite, WebSearch, WebFetch, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
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

```
Agent({
  description: "API security audit: <scope>",
  subagent_type: "api-expert:api-expert",
  model: "opus",
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

WALK OWASP API TOP 10 2023 ITEM-BY-ITEM. For each, document:
- Status (Pass / Fail / Partial / N/A)
- Evidence (file:line references or absence of controls)
- Severity if Fail (CRITICAL / HIGH / MEDIUM / LOW)
- Remediation code block

Read ${CLAUDE_PLUGIN_ROOT}/references/security-hardening.md FIRST, then authentication.md and authorization.md.

OWASP API TOP 10 2023 CHECKLIST:

API1:2023 — Broken Object Level Authorization (BOLA)
API2:2023 — Broken Authentication
API3:2023 — Broken Object Property Level Authorization
API4:2023 — Unrestricted Resource Consumption
API5:2023 — Broken Function Level Authorization (BFLA)
API6:2023 — Unrestricted Access to Sensitive Business Flows
API7:2023 — Server Side Request Forgery (SSRF)
API8:2023 — Security Misconfiguration
API9:2023 — Improper Inventory Management
API10:2023 — Unsafe Consumption of APIs

ADDITIONAL CHECKS:
- Secret management (hardcoded keys, .env in .gitignore, rotation)
- JWT attacks (alg:none, RS256-to-HS256, kid injection, jku/x5u injection)
- Dependency supply chain (npm audit, Snyk, SBOM, Sigstore)
- Logging security (pino redact, no PII/tokens in logs)
- Input validation (Zod/Pydantic at boundaries)

DELIVERABLES:
1. Audit table (one row per OWASP item + each additional check)
2. All CRITICAL + HIGH findings with file:line + fix code
3. Top 5 priorities by risk x effort
4. Compliance mapping (if target specified)
5. Security tooling recommendations (what's missing: ZAP in CI, Semgrep rules, etc.)

PROCEED.
```

## Step 3: Relay the audit

Present the full audit table + CRITICAL/HIGH findings in priority order. Offer to apply fixes one-by-one starting with CRITICAL.

## Never do

- Skip any OWASP Top 10 item — mark N/A with reason, don't omit
- Soften severity — CRITICAL findings are CRITICAL even if the codebase is pre-prod
- Audit without grepping the codebase — OWASP item-by-item walk requires evidence
- Add emojis
- Produce generic advice — every finding cites a specific file:line or "absence of X at Y"
