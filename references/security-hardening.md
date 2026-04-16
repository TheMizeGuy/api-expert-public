# Security Hardening and Threats

## OWASP API Security Top 10 (2023)

| # | Risk | Core mitigation |
|---|---|---|
| API1 | **BOLA (Broken Object Level Authorization)** | Auth check on every data-source access using caller identity + object ID. Use UUIDs, not sequential IDs. Return 404 not 403 for cross-tenant. |
| API2 | **Broken Authentication** | OAuth 2.1/OIDC, short-lived access tokens, refresh rotation, PKCE. Never roll your own. |
| API3 | **BOPLA (Broken Object Property Level Auth)** | Explicit DTO allow-lists per endpoint. Never pass raw DB models to responses. |
| API4 | **Unrestricted Resource Consumption** | Per-endpoint rate limits, payload size caps, query complexity limits (GraphQL), timeouts. |
| API5 | **BFLA (Broken Function Level Auth)** | Enforce RBAC/ABAC at middleware layer, deny by default. |
| API6 | **Unrestricted Access to Sensitive Business Flows** | Rate limiting + CAPTCHA + device fingerprinting on high-value flows. |
| API7 | **SSRF** | URL allowlist, block RFC 1918 + link-local + metadata IPs, resolve DNS once then check. |
| API8 | **Security Misconfiguration** | Automate config hardening. Disable debug endpoints, stack traces, default creds. |
| API9 | **Improper Inventory Management** | API gateway as single entry, version inventory, deprecation policy, shadow API discovery. |
| API10 | **Unsafe Consumption of APIs** | Validate + sanitize third-party API data. Apply timeouts, circuit breakers. |

BOLA is #1 — present in ~40% of API attacks.

## BOLA prevention pattern

```typescript
// Every endpoint taking an object ID MUST verify ownership
async function checkResourceOwnership(req, res, next) {
  const resource = await db.resource.findUnique({
    where: { id: req.params.id },
    select: { ownerId: true, tenantId: true },
  });
  if (!resource) return res.status(404).json({ error: "Not found" });
  if (resource.tenantId !== req.user.tenantId) {
    return res.status(404).json({ error: "Not found" }); // 404, not 403
  }
  next();
}
```

## TLS configuration

- TLS 1.3 only (or 1.2+ with secure cipher suites)
- HSTS with preload: `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`

## CORS hardening

- Allowlist specific origins, NEVER `*` with credentials
- Restrict methods and headers to what's actually needed
- Set `Access-Control-Max-Age` to reduce preflight requests

## Security headers

| Header | Value |
|---|---|
| `X-Content-Type-Options` | `nosniff` |
| `Permissions-Policy` | Restrict APIs (camera, microphone, geolocation) |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Content-Security-Policy` | Restrict sources for any HTML served |

## Input validation

- Boundary-only validation (Zod, Pydantic, etc. at API entry points)
- Parameterized queries everywhere (never string concatenation for SQL)
- HTML sanitization only for HTML output (DOMPurify)

## Secret management

- Never hardcode API keys, passwords, tokens
- `.env` files in `.gitignore`
- Use platform sealed variables (Railway, Vercel, etc.)
- Rotate secrets on schedule (90 days for signing keys)
- Secret scanning: Gitleaks pre-commit + TruffleHog CI

## Supply chain security

- `npm audit` / `pip-audit` / `cargo audit` in CI
- Dependabot or Socket.dev for dependency monitoring
- SBOM generation (CycloneDX or SPDX)
- npm `--provenance` flag for published packages
- Sigstore signing on artifacts

## Logging security

- Configure pino `redact` paths for sensitive fields
- Never log auth tokens, passwords, SSNs, credit cards
- GDPR retention: auth logs 90d, application logs 30d

## Error response hardening

- No stack traces in production responses
- No DB schema leaks in error messages
- Canonical error shape (RFC 9457 Problem JSON)
- Same error shape for all HTTP errors (consistency for clients)

## SSRF prevention

```typescript
// Validate URL before making outbound request
function isAllowedUrl(url: string): boolean {
  const parsed = new URL(url);
  const blockedRanges = ['10.', '172.16.', '192.168.', '169.254.', '127.', '0.'];
  // Resolve DNS first, then check IP
  // Block non-http(s) schemes
  return parsed.protocol === 'https:' &&
    !blockedRanges.some(r => parsed.hostname.startsWith(r));
}
```
