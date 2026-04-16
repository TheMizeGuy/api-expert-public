# Schema Design, Contracts, and Versioning

## Schema languages

| Language | Version | Best for |
|---|---|---|
| **OpenAPI** | 3.1 (full JSON Schema 2020-12) | Public REST APIs, SDK generation, docs |
| **JSON Schema** | 2020-12 | Validation, config schemas, OpenAPI backbone |
| **Protobuf** | proto3 / editions | gRPC, Kafka payloads, internal service-to-service |
| **GraphQL SDL** | Oct 2021 spec | Client-driven queries, federated graphs |
| **Avro** | 1.12 | Event streaming (Kafka + Confluent registry) |

## OpenAPI 3.1 key changes from 3.0

- Full JSON Schema 2020-12 (not a subset)
- `nullable: true` removed — use `type: [string, "null"]`
- `examples` array (plural) replaces singular `example`
- Top-level `webhooks:` field for server-to-client events
- `mutualTLS` security scheme added
- Full `$id`, `$anchor`, `$defs` support

## Runtime validation libraries (TypeScript)

| Library | Bundle (min+gz) | Throughput | Best for |
|---|---|---|---|
| **Zod** | 13.6 KB | 1x baseline | Ecosystem dominance, inference, transforms |
| **Valibot** | 1.0 KB (tree-shaken) | ~2x Zod | Bundle-critical (mobile, edge) |
| **ArkType** | 2.5 KB | ~100x Zod | Hot-path perf, embedded schemas |
| **TypeBox** | 4.2 KB | ~50x Zod | JSON Schema native, Fastify integration |

## Error response contract (RFC 9457 Problem JSON)

```json
{
  "type": "https://api.example.com/errors/insufficient-funds",
  "title": "Insufficient Funds",
  "status": 422,
  "detail": "Account balance is $10.00, but transfer requires $25.00",
  "instance": "/transfers/abc123"
}
```

Content-Type: `application/problem+json`. Always include `type` (URI), `title`, `status`. Optional: `detail`, `instance`, extension fields.

## Pagination contracts

| Pattern | Best for | Tradeoffs |
|---|---|---|
| **Cursor-based** | Default for REST at scale | Stable under concurrent writes, no total count |
| **Relay Connection** | GraphQL | edges/node/pageInfo/cursor standard |
| **Offset/limit** | Simple admin UIs, small datasets | Breaks under concurrent writes, O(n) skip |
| **Keyset** | DB-native (WHERE id > ?) | Fastest, but requires monotonic sort key |

Hard-cap page size (max 100 typical). Never allow unbounded results.

## Idempotency (Stripe pattern)

Client sends `Idempotency-Key` header on POST/PATCH. Server stores key + response for 24h. Replay returns cached response. Use UUIDv7 for keys.

## Versioning strategies

| Strategy | Pros | Cons | Used by |
|---|---|---|---|
| **URL path** (`/v1/`, `/v2/`) | Explicit, cacheable | Fragments codebase + SDKs | Most public APIs |
| **Header** (`Accept-Version`) | Clean URLs | Invisible, harder to cache | Stripe (via custom header) |
| **Stripe evolutionary** | Zero breaking for external clients | Complex internally (version-rewriting middleware) | Stripe, Anthropic |
| **GraphQL versionless** | `@deprecated` + additive evolution | Requires discipline, no clean breaks | GitHub, Shopify |

**Default 2026**: Stripe-style evolutionary for public APIs; URL path for simple internal APIs; GraphQL versionless for GraphQL.

## Protobuf compatibility rules

- NEVER reuse field numbers (even after removing a field — use `reserved`)
- Adding fields is safe; removing requires `reserved`
- Changing field types requires new field number
- `optional` keyword (proto3 2020+) for nullable fields

## Anti-patterns

- Leaking DB models directly through API (tight coupling, security risk)
- Returning all fields by default (use explicit DTOs)
- Sequential integer IDs in URLs (enumeration attack — use UUIDs)
- Missing error schemas in OpenAPI spec
