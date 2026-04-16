# Multi-Platform Client Access

## SDK generation tools (2026)

### Commercial generators

| Tool | Output languages | Notable users | Pricing |
|---|---|---|---|
| **Stainless** | Python, TS, Go, Java, Kotlin, Ruby, Rust, C#, Terraform | OpenAI, Anthropic, Cloudflare | Enterprise |
| **Speakeasy** | TS, Python, Go, Java, C#, Ruby, PHP, Swift, Terraform | Vercel, Mistral | Free tier + Team ~$250/mo |
| **Fern** | TS, Python, Go, Java, C#, Ruby | Cohere, Webflow, ElevenLabs | Free OSS + cloud plans |
| **Kiota** | C#, Go, Java, PHP, Python, Ruby, Swift, TS | Microsoft Graph (12K+ ops) | Free (Microsoft OSS) |
| **OpenAPI Generator** | 50+ languages | Legacy default | Free (OSS) |

### TypeScript-specific

| Tool | Output | Bundle | Best for |
|---|---|---|---|
| **openapi-typescript + openapi-fetch** | Types + tiny client | ~5 KB gzip | Minimum viable SDK, zero runtime bloat |
| **@hey-api/openapi-ts** | Full SDK + TanStack Query hooks | Medium | Full-featured with framework hooks |
| **orval** | SDK + React Query / SWR hooks | Medium | React-heavy projects |

### iOS/Swift

| Tool | Notes |
|---|---|
| **Apple swift-openapi-generator** | Type-safe, protocol-based, Apple-maintained |
| **Kiota (Swift)** | Microsoft-maintained, good for large specs |
| **Speakeasy (Swift)** | Commercial, newer Swift support |

## BFF pattern

One backend per frontend client (web/iOS/Android). The BFF:
- Aggregates multiple service calls into one client-optimized response
- Holds auth tokens server-side (tokens never in browser)
- Shapes payloads per client capability

## Client contract patterns

| Pattern | Use when |
|---|---|
| **REST + SDK** | Public API, external consumers |
| **GraphQL** | Diverse clients needing different data shapes |
| **tRPC** | Same-team TypeScript frontend + backend |
| **Hono RPC** | Hono server + TypeScript client |

## Response filtering

- OpenAPI `fields` parameter for sparse fieldsets
- GraphQL: built-in field selection
- REST: explicit include/exclude parameters

## Pagination per platform

| Platform | Consideration |
|---|---|
| Web | Infinite scroll or paginated table; cursor-based default |
| Mobile | Infinite scroll with prefetch; smaller page sizes (20 vs 50) |
| CLI | Stream all or paginate with `--page-size` flag |

## iOS-specific concerns

- Background fetch: `BGAppRefreshTask`, 30s execution limit
- Widgets (WidgetKit): timeline-based, no real-time
- Live Activities: per-activity push tokens (not device tokens)
- App Intents: Siri/Shortcuts integration needs simplified API surface

## Android-specific concerns

- WorkManager for background tasks (replaces JobScheduler)
- Kotlin Multiplatform for shared networking code
- Battery optimization awareness (doze mode affects connectivity)

## Web SPA concerns

- CORS configuration must match all client origins
- Streaming (SSE, WebSocket) for real-time
- Bundle size: prefer openapi-typescript over full SDK generators

## CLI concerns

- Device code flow (RFC 8628) for auth
- Machine tokens for CI/CD automation
- Streaming output for long operations

## Offline and sync patterns

| Pattern | Complexity | Use when |
|---|---|---|
| **Optimistic updates** | Low | Single-user, conflict-unlikely |
| **CRDTs** | High | Multi-user, conflict-heavy (collaborative editing) |
| **Event sourcing** | Medium | Audit trail needed, temporal queries |
| **Last-write-wins** | Lowest | Acceptable data loss on conflict |

## Webhooks

- HMAC-SHA256 signature verification
- Retry with exponential backoff (3 retries typical)
- Idempotency keys to prevent duplicate processing
- Per-platform delivery: APNs for iOS, FCM for Android, direct HTTP for web
