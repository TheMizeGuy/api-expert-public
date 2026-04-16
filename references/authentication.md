# Authentication and Identity

## OAuth 2.1 (draft-15, 2026)

Consolidates OAuth 2.0 + security BCPs. Key changes:

| Change | Detail |
|---|---|
| PKCE mandatory | All clients (public AND confidential) |
| Implicit grant removed | Use authorization code + PKCE instead |
| ROPC grant removed | Resource Owner Password Credentials gone |
| Exact redirect URI matching | No prefix matching |
| Bearer tokens in URI prohibited | Authorization header or POST body only |
| Refresh token rotation | Public clients MUST use sender-constrained or one-time-use rotation |
| DPoP recommended | Proof-of-Possession for browser-based apps |

## JWT signing algorithms

| Algorithm | Type | When |
|---|---|---|
| HS256 | Symmetric | Single service only. Never distributed. |
| RS256 | Asymmetric RSA | Multi-service: sign with private, verify via JWKS |
| ES256 | ECDSA P-256 | Smaller signatures, recommended for new systems |
| EdDSA (Ed25519) | Asymmetric | Fastest modern signing |

## Critical JWT validation rules

EVERY consumer MUST verify: `exp`, `iss`, `aud`, `nbf`, signature (against expected algorithm and key only).

## JWT attack vectors

| Attack | Mechanism | Mitigation |
|---|---|---|
| `alg:none` | Strip signature, set alg to none | Whitelist allowed algorithms at library level |
| RS256-to-HS256 | Sign with public RSA key as HMAC secret | Enforce asymmetric validation, separate key types |
| `kid` path traversal | kid as file path -> attacker controls key | Never use kid in filesystem ops, lookup against allowlist |
| `kid` SQL injection | kid in unparameterized query | Parameterize all queries |
| `jku`/`x5u` injection | Attacker provides URL to own key set | Pin to trusted URLs, prefer JWKS discovery |

## Session management

- BFF pattern for browser apps: tokens never in browser, session cookie only
- Cookie flags: `HttpOnly`, `Secure`, `SameSite=Lax` (or `Strict`)
- Session ID regeneration on privilege change
- Idle timeout + absolute timeout

## Device code flow (RFC 8628)

For CLIs, smart TVs, constrained devices. Device shows user_code + URI; user authorizes on secondary device; device polls token endpoint.

## WebAuthn / passkeys

- Phishing-resistant (origin-bound credentials)
- No shared secrets (asymmetric key pair per credential)
- Discoverable credentials = passwordless login
- Platform authenticators (Touch ID, Face ID, Windows Hello) + roaming (YubiKey)

## Identity providers (2026)

| Provider | Strength |
|---|---|
| Auth0 (Okta) | Broadest feature set, Actions for customization |
| WorkOS | Enterprise SSO + directory sync + AuthKit UI |
| Clerk | DX-first, pre-built components, fastest time-to-auth |
| Supabase Auth | Postgres RLS integration, GoTrueV2 |
| Firebase Auth | Google ecosystem, generous free tier |

## Key rotation

Rotate signing keys every 90 days. Publish both old and new keys in JWKS during transition. Set `kid` on every token.

## Password hashing

Use argon2id with OWASP recommended params. NEVER sha256/md5.

## Rate limiting at auth endpoints

Per-IP + per-account rate limits on `/login`, `/register`, `/forgot-password`. Throttling preferred over lockout (NIST).

## Service-to-service auth

- mTLS with SPIFFE/SPIRE for workload identity
- Machine-to-machine OAuth (`client_credentials` grant)
- API keys for simple internal services (with rotation policy)

## Device attestation

- iOS: App Attest (DeviceCheck framework)
- Android: Play Integrity API
- Both: verify server-side, don't trust client assertions alone
