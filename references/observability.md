# Observability and Operations

## Three pillars

| Pillar | Answers | Storage |
|---|---|---|
| **Logs** | What happened (discrete events) | Loki, Elasticsearch, Datadog |
| **Metrics** | How much / how fast (time series) | Prometheus, VictoriaMetrics, Mimir |
| **Traces** | Where did the request go (causal graph) | Tempo, Jaeger, Honeycomb |

OpenTelemetry unifies all three with one SDK + Collector pipeline.

## Structured logging libraries

### Node.js

| Library | Throughput | Status |
|---|---|---|
| **pino** | 222K ops/sec | Active, 5-8x faster than winston |
| **winston** | 36K ops/sec | Community-only, synchronous serialization |

**2026 default**: pino. Log to stdout, let infrastructure route. Use pino-http for request middleware.

### Python

**structlog** (25.5.0) — processor pipeline, ConsoleRenderer dev / JSONRenderer prod, asyncio-native.

### Go

**slog** (stdlib, Go 1.21+) — default for new projects. zerolog for absolute high-throughput.

## Essential log fields per API request

| Field | Purpose |
|---|---|
| `requestId` / `traceId` | Correlation across services |
| `method`, `path`, `statusCode` | Request identification |
| `duration` | Latency tracking |
| `userId`, `tenantId` | Attribution (redact in external logs) |
| `error` (structured) | Machine-parseable error info |

## Log levels

`TRACE < DEBUG < INFO < WARN < ERROR < FATAL`

INFO for business milestones. ERROR for failed operations. Never use WARN for expected conditions.

## OpenTelemetry

- **OTLP**: Wire protocol for all signals
- **W3C Trace Context**: `traceparent` + `tracestate` headers propagate across services
- **Sampling**: Head-based (decide at trace start) or tail-based (decide after trace completes)
- **Auto-instrumentation**: available for Node.js, Python, Java, Go, .NET

## Metrics: RED and USE methods

| Method | Metrics | Use for |
|---|---|---|
| **RED** (Request, Error, Duration) | Rate, Error rate, Duration (p50/p95/p99) | API endpoints |
| **USE** (Utilization, Saturation, Errors) | CPU%, queue depth, error count | Infrastructure resources |

## Cardinality warning

**Never** use user_id, request_id, or other high-cardinality values as metric labels. 150M+ series explosions are real and will crash Prometheus.

## SLO + error budget

| Concept | Definition |
|---|---|
| **SLI** | Measured ratio (e.g., successful requests / total requests) |
| **SLO** | Target (e.g., 99.9% success rate over 30 days) |
| **Error budget** | 1 - SLO (e.g., 0.1% = 43.2 min/month of allowed downtime) |

### Multi-window multi-burn-rate alerts (Google SRE Workbook Ch. 5)

| Window | Burn rate | Alert |
|---|---|---|
| 1h + 5m | 14.4x | Page (immediate) |
| 6h + 30m | 6x | Page (sustained) |
| 3d + 6h | 1x | Ticket (slow burn) |

## Long-term metrics storage

| Backend | Strength |
|---|---|
| **Prometheus** | Standard, PromQL, short-term (15d-30d) |
| **VictoriaMetrics** | 10x compression vs Prometheus, long-term |
| **Mimir (Grafana)** | Horizontally scalable, multi-tenant, S3-backed |

## Error tracking

**Sentry** — de facto standard for application error tracking. Source maps, breadcrumbs, release tracking, performance monitoring.

## Incident response

- PagerDuty or Grafana Cloud IRM for on-call
- Blameless postmortems (focus on systems, not people)
- Document: timeline, impact, root cause, action items, prevention
