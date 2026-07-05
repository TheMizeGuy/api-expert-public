# Changelog

All notable changes to this plugin are documented here. Format loosely follows
[Keep a Changelog](https://keepachangelog.com/).

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
