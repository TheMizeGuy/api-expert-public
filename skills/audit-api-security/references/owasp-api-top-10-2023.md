# OWASP API Top 10 2023 — canonical audit checklist

Currency note (verified 2026-07): the 2023 edition remains the latest API-specific Top 10. The
OWASP web-app Top 10:2025 is a SEPARATE list — do not conflate the two or "upgrade" this checklist
against it.

Canonical checklist for the api-expert plugin. Owned by `skills/audit-api-security` (its briefing
passes this file's path to the dispatched agent); `skills/review-api` links here for its OWASP
coverage row, and `agents/api-expert.md` points here for direct-dispatch security work. No other
copy of this checklist exists in the plugin — edit it here only.

Walk EVERY item. For each, record: Status (Pass / Fail / Partial / N/A), Evidence (file:line or the
absence of a control at a named location), Severity if Fail or Partial (decision test below), and a
remediation code block.

## The ten items

### API1:2023 — Broken Object Level Authorization (BOLA)
- Grep for endpoints taking object IDs in path/body
- Verify each performs ownership check before returning object
- Common pattern to find: `.findById(id)` without `AND user_id = ?`
- ~40% of API attacks exploit this [P — OWASP 2023]

### API2:2023 — Broken Authentication
- JWT validation: aud + iss + exp + signature + alg whitelist (not reading from token header)
- Session: cookie flags (HttpOnly, Secure, SameSite), session ID regeneration on privilege change
- Rate limiting on /login, /register, /forgot-password (per-IP + per-account)
- MFA availability for privileged accounts
- Password hashing (argon2id OWASP params, NEVER sha256/md5)
- Account lockout vs throttling (throttling preferred, NIST)

### API3:2023 — Broken Object Property Level Authorization
- Mass assignment: endpoints that accept arbitrary object properties
- Field-level authorization on sensitive props (isAdmin, role, balance)
- Response filtering: don't leak internal fields

### API4:2023 — Unrestricted Resource Consumption
- Rate limiting present (token bucket / GCRA)
- Per-IP + per-user + per-tenant layers
- File upload size limits
- Query complexity limits (GraphQL)
- Pagination enforcement (hard cap on page size)
- 429 + Retry-After + X-RateLimit-* headers

### API5:2023 — Broken Function Level Authorization (BFLA)
- Admin endpoints reachable by regular users? (grep for admin middleware)
- Consistent RBAC/ABAC enforcement across endpoint groups
- Intentional `deny-by-default` not `allow-by-default`

### API6:2023 — Unrestricted Access to Sensitive Business Flows
- CAPTCHA / bot detection on high-value flows (signup, checkout, password reset)
- Anti-automation on enumeration-prone endpoints

### API7:2023 — Server Side Request Forgery (SSRF)
- URL inputs validated against allowlist
- Internal metadata endpoints (169.254.169.254) blocked
- DNS rebinding protection

### API8:2023 — Security Misconfiguration
- TLS 1.3 only (or 1.2+ with secure ciphers)
- HSTS with preload
- CORS: allowlist origins, no wildcard + credentials
- CSP headers (for any HTML served)
- Security headers: X-Content-Type-Options, Permissions-Policy, Referrer-Policy
- Error responses: no stack traces, no DB schema, canonical error shape (RFC 9457)
- Default credentials removed
- Unused endpoints / admin panels disabled in prod

### API9:2023 — Improper Inventory Management
- Inventory of all APIs (catalog, OpenAPI in git, dev portal)
- Shadow API detection (undocumented endpoints)
- Zombie API detection (deprecated but live)
- Deprecated versions have sunset dates

### API10:2023 — Unsafe Consumption of APIs
- Third-party API calls validated (don't trust response shape)
- Timeouts on outbound calls
- Retries with limits
- Response schema validation

## Additional check groups (all five are audit-table rows)

### Secret management
- grep for hardcoded API keys, passwords, tokens
- Verify .env in .gitignore
- Check secret rotation mechanism
- Sealed/encrypted variables on the deployment platform in use

### JWT-specific attacks
- alg:none attack (library explicitly whitelists algorithms?)
- RS256→HS256 confusion (separate verify functions per alg class)
- kid injection (kid used in path or SQL without sanitization)
- jku / x5u injection (header URLs pinned)

### Dependency supply chain
- npm audit / pip-audit / cargo audit clean?
- Known CVEs in dependencies?
- socket.dev / Snyk / Dependabot active?
- SBOM generated (CycloneDX or SPDX)?
- Sigstore signing on published artifacts?
- Secret scanning (Gitleaks pre-commit + TruffleHog CI)?

### Logging security
- pino `redact` paths configured for sensitive fields
- structlog processors scrubbing PII
- No auth tokens, passwords, SSNs, credit cards in logs
- Retention policy aligned with applicable regulation (e.g. GDPR)

### Input validation
- Boundary-only validation (Zod / Pydantic / etc.)
- HTML sanitization (DOMPurify — but only for HTML output, never SQL)
- Parameterized queries everywhere

## Severity decision test

Assign severity to every Fail/Partial with this ordered test — first match wins:

1. **CRITICAL** — exploitable pre-auth; OR an authenticated user can read/write another tenant's or
   another user's data (cross-boundary BOLA/BFLA); OR secrets/credentials exposed (hardcoded,
   logged, or leaked in error responses); OR authentication bypassable entirely (alg:none accepted,
   unsigned JWT verified, default credentials live).
2. **HIGH** — an authenticated user can exceed their own privilege within their tenant; OR no rate
   limiting on auth endpoints (credential stuffing viable); OR a resource-consumption attack can
   take the service down; OR a dependency carries a known-exploited CVE on a reachable code path.
3. **MEDIUM** — control present but partial or inconsistent (some endpoints covered, others not);
   OR misconfiguration with no direct exploit path today (missing HSTS preload, permissive but
   non-wildcard CORS); OR missing inventory/telemetry controls (API9 gaps).
4. **LOW** — hardening gaps fully shielded by other controls; header hygiene on non-HTML API
   responses; documentation/spec drift with no security consequence.

Modifiers:
- A named compliance target raises severity one level when the finding blocks a specific
  requirement (e.g., under PCI-DSS, card data in logs is CRITICAL regardless of exploitability).
- Pre-prod environment NEVER lowers severity — the code ships as-is.

## Worked finding example (imitate this form exactly)

Audit-table row:

| Item | Status | Evidence | Severity | Remediation |
|---|---|---|---|---|
| API1 BOLA | Fail | `src/routes/orders.ts:42` — `Order.findById(req.params.id)` returns any order; no ownership predicate anywhere in the handler or middleware chain | CRITICAL (decision test rule 1: authenticated user reads another user's data) | see code block below |

Remediation code block accompanying the row:

```ts
// src/routes/orders.ts:42 — scope the lookup to the authenticated owner
const order = await Order.findOne({
  where: { id: req.params.id, userId: req.auth.userId },
});
if (!order) return problem(res, 404, "Order not found");
// 404, not 403: a 403 confirms the order ID exists and enables enumeration
```

Every finding needs all four parts: a status, evidence at file:line (or the named absence of a
control), a severity justified by the decision-test rule number, and runnable remediation code.
"Add an ownership check" without code is not a finding — it is advice.
