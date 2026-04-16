# Documentation, DX, and Lifecycle

## Documentation tools (2026)

| Tool | Strength | Price |
|---|---|---|
| **Redoc** | Polished 3-panel OpenAPI renderer | Free (OSS) |
| **Scalar** | Modern renderer, interactive Try It, multi-language code gen | $12/user/mo hosted |
| **Mintlify** | AI-first, MDX, MCP server per docs site | $0-$300/mo |
| **Stoplight** | Design workflow + mock server + governance | $49/user/mo |
| **ReadMe** | Live API Reference with real requests, usage analytics | $99/mo+ |
| **Redocly Realm** | Redocly CLI OSS + hosted docs | $10/user/mo |

**Mintlify** ships an MCP server per docs site — AI tools connect directly to docs. Nearly half of portal traffic from AI agents.

## API style guides

**Zalando RESTful API Guidelines** — encode as Spectral rules for automated linting.

Key principles:
- Use kebab-case for URIs, camelCase for JSON properties
- Use RFC 9457 Problem JSON for errors
- Use cursor-based pagination
- Use ISO 8601 for dates
- Use plural nouns for collections

## Versioning strategies

| Strategy | Used by | Key trait |
|---|---|---|
| **Stripe date-versioning** | Stripe, Anthropic | Pin version per API key; version-rewriting middleware translates |
| **URL path** (`/v1/`) | Most public APIs | Explicit but fragments codebase |
| **Shopify quarterly** | Shopify | Quarterly named releases, 12-month support |
| **GitHub 24-month** | GitHub | Longest deprecation notice in industry |
| **GraphQL versionless** | GitHub, Shopify (GraphQL) | Additive evolution + @deprecated |

## Deprecation and sunset (RFC 9745 + RFC 8594)

### HTTP headers

```
Deprecation: @1735689600
Link: <https://developer.example.com/migrate/v1-to-v2>; rel="deprecation"

Sunset: Tue, 31 Dec 2026 23:59:59 GMT
Link: <https://developer.example.com/sunset/v1>; rel="sunset"
```

**Sunset MUST NOT be earlier than Deprecation.**

### Timeline

1. Announce deprecation (Deprecation header + changelog + email)
2. 12-month minimum notice for external (24 months GitHub-style)
3. Publish migration guide
4. Track usage telemetry
5. Proactive outreach to top users
6. Enforce sunset: 410 Gone + RFC 9457 body
7. Grace period: serve 410 for 30-90 days
8. Remove handler (404)

### Spec markers

- OpenAPI: `deprecated: true` + `x-sunset` extension
- GraphQL: `@deprecated(reason: "Use X instead. Sunset YYYY-MM-DD.")`
- TypeScript: `/** @deprecated Use X instead. Sunset YYYY-MM-DD. */`

## Changelog format

**Keep a Changelog** format + **Conventional Commits** (`feat:`, `fix:`, `BREAKING CHANGE:`).

```markdown
## [v1.8.0] - 2026-04-14
### Deprecated
- `GET /v1/users/{id}/profile` — use `GET /v2/users/{id}` instead. Sunset 2026-12-31.
```

## Release automation

| Tool | Best for |
|---|---|
| **semantic-release** | Single-package repos, fully automated |
| **changesets** | Monorepos, human-in-the-loop |

## SDK publishing

- npm: `--provenance` flag for Sigstore provenance
- PyPI: Trusted Publishers (GitHub Actions OIDC)
- crates.io: Trusted Publishers
- All: sign with Sigstore, generate SBOM

## Spectral linting

Enforce API style guides as code. Run in CI alongside tests.

```yaml
# .spectral.yaml
extends: spectral:oas
rules:
  operation-operationId: error
  oas3-api-servers: error
  contact-properties: warn
```

## DX metrics

| Metric | Measures |
|---|---|
| **Time to First Call (TTFC)** | How fast a new developer makes their first successful API call |
| **Time to First Hello World** | Full onboarding to working integration |
| **API Error Rate** | 4xx/5xx as experienced by consumers |
| **SDK Adoption** | Downloads, active keys, version distribution |
