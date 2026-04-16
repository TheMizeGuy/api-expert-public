# Performance, Rate Limiting, and Caching

## Rate limiting algorithms

| Algorithm | Burst behavior | Memory/client | When |
|---|---|---|---|
| **Token bucket** | Allows bursts up to capacity | ~16 bytes | General-purpose default (Stripe uses this) |
| **GCRA** | Single TAT timestamp | 1 key per entity | Per-user limits at massive scale (Cloudflare uses this) |
| **Sliding window counter** | Weighted formula | 32 bytes (dual keys) | Best accuracy + memory balance |
| **Fixed window** | Simple counter | 16 bytes | DANGER: allows 2x limit at boundaries |
| **Sliding window log** | Exact timestamps | 8 bytes x N | When precise history matters |

**Default**: Token bucket for general-purpose. GCRA for per-user at scale.

## Redis rate limiting

All implementations use Lua scripts for atomic read-modify-write (no TOCTOU races, single round trip).

## Rate limit response headers

| Header / status | Purpose |
|---|---|
| `429 Too Many Requests` | Quota exceeded |
| `Retry-After: <seconds>` | Client must wait |
| `RateLimit-Limit` | Total quota |
| `RateLimit-Remaining` | Remaining quota |
| `RateLimit-Reset` | When quota resets |

## HTTP caching (RFC 9111)

| Directive | Effect |
|---|---|
| `Cache-Control: public, s-maxage=3600` | CDN caches for 1h |
| `stale-while-revalidate=60` | Serve stale while revalidating in background |
| `ETag` + `If-None-Match` | Conditional GET, 304 Not Modified |
| `Vary: Accept, Authorization` | Cache varies by these headers |

## CDN edge caching

| CDN | Strength |
|---|---|
| **Cloudflare** | Tiered Cache, Workers for edge compute |
| **Fastly** | VCL/Compute@Edge, real-time purge |
| **Vercel** | Next.js-native ISR, edge functions |

## Application caching (tiered)

- **L1 (in-process)**: `lru-cache` / `node-cache` — sub-ms, per-instance
- **L2 (Redis)**: shared across instances, ~1ms network
- **Cache stampede prevention**: XFetch (probabilistic early expiration) or single-flight (only one caller recomputes)

## Database connection pooling

| Pooler | Peak TPS | Sweet spot |
|---|---|---|
| **PgBouncer** | 44K | Lowest latency at low connection counts |
| **pgcat** | 59K | Multi-threaded Rust, outperforms past 50 clients |
| **Supavisor** | — | Supabase-native, Elixir-based |

**Pool sizing formula** (HikariCP): `connections = (core_count * 2) + spindle_count`

**Transaction mode**: Use PgBouncer/pgcat in transaction mode. Protocol-level prepared statements supported in PgBouncer 1.21+.

Set timeouts: `statement_timeout`, `lock_timeout`, `idle_in_transaction_session_timeout`.

## N+1 prevention

| Tool | Approach |
|---|---|
| **DataLoader** | Batches + caches within single request |
| **Prisma `include`** | Eager loading via joins |
| **Drizzle `with`** | Similar eager loading |

## Cache key design

- Deterministic: sorted query params
- Version prefix: `v2:users:123:profile`
- Separate keys per tenant

## Performance optimization workflow

1. **Measure first**: pg_stat_statements (ORDER BY total_exec_time DESC), OTel traces (p95/p99 by span), k6 profile under load
2. **Rank by impact x effort**: High impact + Low effort = fix first
3. **Fix one thing at a time**: Measure after each change
4. **Common bottlenecks**: Missing indexes, N+1 queries, connection pool exhaustion, cold cache, downstream timeouts

## Request hedging

Send duplicate request to a second replica at P80 delta. AWS case study: 29% p99 latency reduction, 8% duplicate rate (optimal tradeoff).
