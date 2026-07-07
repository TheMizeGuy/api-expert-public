# Deployment & Infrastructure

Deployment-layer concerns for API audits and designs. Platform-neutral — examples name Railway,
Fly.io, Render, and Kubernetes where a concrete anchor helps, but every check applies to any host.

## Health checks

- Separate LIVENESS (process is up — restart me if not) from READINESS (safe to receive traffic).
  A liveness probe that touches the database turns a DB blip into a restart storm.
- Readiness should verify critical dependencies (DB reachable, migrations applied, cache warm
  enough) and flip to unready during shutdown BEFORE the process stops accepting connections.
- The health endpoint must be unauthenticated, cheap, and excluded from rate limiting and access
  logs (or sampled) — platforms poll it constantly.
- Startup probes (or platform healthcheck timeouts) must exceed the real cold-start time,
  including migration time if migrations run on boot.

## Zero-downtime deploys

- Baseline: rolling replacement with overlap — new instance passes readiness before the old one is
  drained. Verify the platform actually overlaps (some restart-in-place by default).
- Blue-green: full parallel environment, atomic traffic cutover, instant rollback. Costs double
  capacity during the window; pairs well with the migration patterns below.
- Canary: shift a traffic percentage to the new version and compare error rate/latency before
  full rollout. Needs per-version metrics to mean anything.
- Preview environments per PR (Railway PR envs, Fly/Render preview apps, k8s namespaces) catch
  integration issues before main — flag their absence for teams shipping API contract changes.

## Graceful shutdown

- Handle SIGTERM: stop accepting new connections, finish in-flight requests (bounded by a drain
  timeout), close DB pools and queue consumers, then exit. Killing mid-request surfaces to
  clients as socket resets — indistinguishable from an outage.
- Deregister from the load balancer / flip readiness BEFORE draining; otherwise traffic keeps
  arriving during the drain window.
- Long-lived connections (WebSocket, SSE, gRPC streams) need explicit shutdown semantics:
  GOAWAY / close frames with a reconnect contract, not silent drops.
- Background workers and cron consumers need the same treatment — an interrupted job must be
  resumable or idempotent.

## Config and secrets at the deploy layer

- Secrets live in the platform's secret store (sealed variables, secret manager), never in the
  image, the repo, or plain env files committed anywhere.
- Rotation must be possible without a code change — inject via env/secret mounts, read at boot
  (or hot-reload where the platform supports it).
- Config drift check: the deployed environment's variables should be reproducible from
  config-as-code (railway.toml / fly.toml / manifests), not hand-edited dashboard state.
- Audit check: who can read production secrets, and is that surface logged?

## Scaling

- Horizontal scaling requires statelessness: no in-process sessions, no local file writes that
  matter, no in-memory caches treated as authoritative. Flag sticky-session dependencies.
- Connection-bound services (Postgres max_connections) often need a pooler (PgBouncer or the
  platform equivalent) BEFORE adding replicas — more replicas without pooling exhausts the DB.
- Vertical scaling is the right first move for memory-bound or single-hot-path services; note it
  as a deliberate choice, not a failure to scale out.
- Autoscaling on the wrong signal (CPU for an IO-bound API) oscillates; scale on concurrency or
  queue depth where the platform allows.

## Migrations on deploy

- Expand-contract: additive migration first (new column/table, nullable), deploy code that writes
  both, backfill, deploy code that reads new, then contract in a LATER release. Never destructive
  DDL in the same release as the code that depends on it.
- Decouple "deploy" from "migrate" when possible: a migration job/step that runs once, not N
  times for N replicas racing on boot. Advisory locks or the platform's release-phase mechanism
  prevent concurrent migration runs.
- Every migration needs a rollback story — and "restore from backup" is a recovery plan, not a
  rollback plan.

## Restart policies and crash loops

- Restart policy with backoff, plus an alert on restart frequency — a service that restarts every
  90 seconds is down, even though it reports "running".
- Crash-loop triage order: config/secret missing → migration failure → port binding (wrong
  interface: bind `::` or `0.0.0.0`, not localhost, inside containers) → OOM (check limits).

## Audit checklist (what to verify in a deployment scope)

- Health endpoint exists, is cheap, distinguishes liveness from readiness
- Deploys overlap old/new instances; rollback procedure documented and tested
- SIGTERM drain implemented; drain timeout > p99 request duration
- No secrets in repo/image; rotation possible without redeploy of code
- Config reproducible from config-as-code
- Pooling in place before horizontal DB-backed scaling
- Migrations expand-contract, single-runner, with rollback notes
- Restart-frequency alerting exists
