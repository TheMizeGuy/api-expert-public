# Changelog

All notable changes to this plugin are documented here. Format loosely follows
[Keep a Changelog](https://keepachangelog.com/).

## [0.2.0] - 2026-07-07

### Fixed

- `api-team-lead` wave-sizing contradiction: the dispatch step and "Rate limit safety" carried
  different concurrency numbers. One canonical statement now lives in Rate limit safety; waves of
  2-4 are a session-reset fallback only, never the default.
- README's version line still read 2.0.0 after the 0.1.2 realignment — now tracks the plugin
  version.
- Three skill descriptions used "comprehensive" — a word the agent's own MUST-NOT list bans as AI
  slop. Reworded.

### Added

- Canonical goodmem retrieve call shape in `agents/api-expert.md` (object-array `space_keys`,
  `fetch_memory: false`) — prevents the most common silent-failure misuse for users who configure
  goodmem.
- OpenAPI 3.2 currency (released 2025-09-18, backward-compatible): the spec-generation workflow
  and `create-api-spec` default NEW specs to 3.2 when toolchain support is verified via context7,
  else 3.1; existing 3.1 specs stay 3.1.
- OWASP currency note in the canonical checklist: 2023 remains the latest API-specific Top 10; the
  web-app Top 10:2025 is a separate list — do not conflate.
- Fix-mode briefing contract in `review-api` (approved findings only, tests run after, per-finding
  applied/skipped report) — the skill offered fix mode but never defined it.
- `api-team-lead`: honors the briefing's SCOPE FILTER; writes the unified report to the briefing's
  FILE path (final messages truncate ~60KB — the file is the durable deliverable).
- Router fast path for factual one-liner questions (inline reference read, no dispatch ceremony)
  and a broader endpoint-detection grep (NestJS decorators, Go HandleFunc, Rails, Django, Axum).

### Changed

- `create-api-spec` response guidance: each operation documents its real success code plus errors
  it can actually return, instead of stamping ten statuses on every operation (spec bloat).
- `deprecate-api` date-format clarification: ISO dates in the plan; the `Sunset` header renders as
  an HTTP-date.

## [0.1.2] - 2026-07-05

### Changed

- **Version realignment**: reset this public mirror's version from 2.0.0 to 0.1.2 so it tracks the private source of truth (`TheMizeGuy/api-expert`). Public mirror versions now move in lockstep with the private plugin; the earlier independent public-milestone numbering (2.0.0) is retired. No functional or content change — this bump only aligns the version number.

## [2.0.0] - 2026-07-05

### Changed (breaking)

- **Model policy**: removed every `model: "opus"` pin — from both agent frontmatter and every
  example `Agent()` dispatch block in the skill files, the README, and the plugin description.
  Both agents (`api-expert`, `api-team-lead`) now inherit the session model — always the strongest
  available Claude at dispatch time — rather than being locked to a specific, eventually-dated
  model. If you had automation depending on the literal string `model: "opus"` appearing in these
  files, update it.
- `cross-project-api-audit` no longer dispatches `api-team-lead` via
  `subagent_type: "api-expert:api-team-lead"` — that silently strips the `Agent` tool the team lead
  needs to fan out sub-agents (a Claude Code platform limitation). It now reads the agent file's
  body and dispatches it under `subagent_type: "general-purpose"`. The README's direct-dispatch
  example was fixed to match, with a warning inline.

### Added

- Optional goodmem (memory) and serena (semantic code navigation) MCP integration across both
  agents and all 10 skills. Every workflow explicitly frames these as "if configured" — nothing
  blocks or degrades if you don't have them. goodmem space IDs in examples are placeholders.
- Canonical OWASP API Top 10 2023 checklist at
  `skills/audit-api-security/references/owasp-api-top-10-2023.md` — all ten items with per-item
  checks, five additional check groups, a severity decision test, and a worked finding example.
  `audit-api-security` and `review-api` both read it directly instead of re-deriving the Top 10 from
  the model's memory.
- Acceptance criteria and a verify-then-relay step on every skill that dispatches an agent — a
  report failing its own stated criteria (missing checklist rows, ungraded findings, unstated
  reasons for N/A) gets re-queried before it reaches you, not relayed as complete.
- Tie-break rules in `api-expert` (the router skill) for requests that match multiple workflow
  keywords (e.g., "review" + "secure" → security workflow wins).
- Worked hypothesis-testing example in `debug-api` to make "close every hypothesis, confirmed or
  refuted" concrete rather than aspirational.
- `USAGE.md` — quickstart, one worked walkthrough per major skill, and a troubleshooting table.
- "Execution mode" note on every dispatching skill: if the session model is already the strongest
  tier and the task is important or complicated, the skill may run its workflow inline instead of
  dispatching a separate agent.

### Fixed

- README component tables, install section, and dependency table now match the actual tree
  (agent/skill counts, reference file list, optional-vs-required dependencies).
- Removed private-topology assumptions from generic guidance (e.g., a specific deployment platform's
  config format in `design-api` and `optimize-api`) in favor of platform-neutral phrasing.

## [1.0.0] - 2026-05-24

- Initial public release: `api-expert` and `api-team-lead` agents, 10 skills, embedded `references/`
  knowledge base (architecture patterns, schema design, authentication, authorization, security
  hardening, client access, performance/caching, observability, testing, inter-service
  communication, documentation lifecycle).
