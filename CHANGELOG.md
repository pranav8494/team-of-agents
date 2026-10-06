# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Added
- `skill-reflector` skill — reviews learnings the orchestrator captured from your corrections, lets you
  approve each as project or global, and offers a PR once a learning has been approved three times
- Session-start hook loads approved learnings from `.team-of-agents/approved-learnings.json` (project)
  and `~/.claude/plugins/team-of-agents/approved-learnings.json` (global); project entries win on id
- If learnings exist but `jq` is not installed, the hook shows a warning with the install command and
  asks Claude to offer the install

### Fixed
- Orchestrator learning capture used `CLAUDE_PROJECT_ROOT`, which Claude Code does not set, so it tried
  to write to `/.team-of-agents`. It now resolves the project root from `CLAUDE_PROJECT_DIR`, the git
  top level, or the working directory, and appends via a quoted heredoc so apostrophes are safe

---

## [3.0.0] - 2026-08-11

Breaking. The frontend skills are replaced by a composable trio plus an overlay mechanism, so domain
and stack knowledge is inherited rather than copied into a forked skill.

### Migration

| Was | Now |
|---|---|
| `/frontend-designer` | `/frontend-engineer` |
| `/frontend-code-reviewer` | `/frontend-reviewer` |
| `/fintech-frontend-engineer` | Unchanged name; now a shim over `frontend-engineer` + `overlays/domains/fintech.md` |

### Added
- `frontend-planner` skill and agent — recon, conventions capture, and task breakdown. Writes
  `FRONTEND-CONVENTIONS.md` when a repo has none, marking each rule observed or proposed
- `frontend-engineer` skill and agent — implementation, modular component contract, data integration
- `frontend-reviewer` skill and agent — PR review and repo audit against the repo's own conventions
- `overlays/domains/` and `overlays/stacks/` — composable knowledge on two axes, each file structured
  as `## Plan deltas` / `## Build deltas` / `## Review deltas`, one section per base skill
- `overlays/domains/fintech.md` — money display, sensitive data, compliance rules, carried over from
  the former `fintech-frontend-engineer` skill
- `overlays/stacks/expo-universal.md` — Expo SDK 53 / React Native 0.79 / React 19 /
  react-native-web / Expo Router, version-stamped and marked verify-before-use
- `_TEMPLATE.md` in both overlay directories
- Overlay-load rule in every frontend skill's Iron Law: load applicable overlays, announce which were
  loaded or state "no overlay", never invent domain or stack rules

### Changed
- `fintech-frontend-engineer` skill and agent reduced to shims over the base plus the fintech overlay
- Orchestrator: routing table, disambiguation table, agent-name list, frontend rule, and quick
  reference updated for the new trio; dispatches should now name the domain and stack
- README specialist tables, usage examples, and repository structure

### Removed
- `frontend-designer` skill and agent, superseded by `frontend-engineer`
- `frontend-code-reviewer` skill and agent, superseded by `frontend-reviewer`

---

## [2.1.0] - 2026-05-06

### Added
- `Iron Law` section to all 18 skills, a concrete, role-specific non-negotiable principle
- `Task Approach` table to all 17 specialist skills, maps task type to what Claude should produce
- `Output Protocol` section to all 17 specialist skills, defines `CONFIDENCE: [High|Medium|Low]` and `BLOCKED:` signal contract
- `version: 2.1.0` in frontmatter of all 18 skills
- `skills` manifest array in `.claude-plugin/plugin.json` with path and version per skill (dynamic capability discovery)
- Specialist disambiguation table in orchestrator, 14-row lookup resolving overlapping specialist boundaries
- Fallback and escalation section in orchestrator, decision table and escalation order
- Context Envelope format in orchestrator Phase 4, standard five-field structure for every agent dispatch (TASK / CONTEXT / CONSTRAINTS / OUTPUT FORMAT / CONFIDENCE SIGNAL)
- Trust Tier Model in orchestrator, T1 research, T2 artifact, T3 execution with required confirmation levels
- Confidence routing in orchestrator Phase 5, orchestrator acts on High / Medium / Low / BLOCKED signals
- BLOCKED signal handling in orchestrator Fallback, three-branch resolution
- Shared Findings Scratchpad in orchestrator Phase 4, lightweight shared-memory mechanism for parallel agent runs
- Agent Trace Log in orchestrator Phase 5 synthesis format
- Optional Critic Pass (Phase 5.5) in orchestrator, conditions table and dispatch envelope for invoking `senior-engineer` as a reviewer
- Timeout and partial-failure recovery in orchestrator Fallback, retry protocol and multi-agent partial-completion rule
- `technical-business-analyst` added to orchestrator routing tables and specialist disambiguation

### Changed
- All 17 specialist skills rewritten from persona-based ("You are a...") to action-oriented (task type → output). Removed `## Your Workflow` (day-in-the-life checklists) from all skills
- SRE skill fully rewritten: removed on-call staffing targets and incident coordination content; replaced with Task Approach table covering SLO definition, observability design, alert rules, PRR, runbooks, postmortems, toil audits, IaC, and chaos experiment design
- Orchestrator routing table updated with dynamic discovery note pointing to `plugin.json` as source of truth

---

## [2.0.2] - 2026-04-03

### Fixed
- Version bump in `marketplace.json` and `package.json` to align with `plugin.json` (both were still on 2.0.2 after the 2.1.0 bump in plugin.json)

---

## [2.0.1] - 2026-04-03

### Added
- **Technical Business Analyst** (`skills/technical-business-analyst/`), bridges stakeholder intent and engineering execution; scope definition, BRDs, functional specs, implementation plans, and requirements traceability

---

## [2.0.0] - 2026-04-02

### Breaking Changes
- Renamed `team` skill to `orchestrator`, invocation changes from `/team-of-agents:team` to `/team-of-agents:orchestrator`

### Added
- **Orchestrator** (`skills/orchestrator/`), full planning and dispatch layer; breaks tasks into subtasks, assigns specialists, dispatches subagents in parallel or sequentially, synthesises results
- **`agents/` directory**, 15 subagent definitions for all specialists; dispatched by the orchestrator using Claude Code's Agent tool
- **Project Manager** (`skills/project-manager/`, `agents/project-manager.md`), delivery and execution specialist; sprint planning, RAID logs, milestones, status reports. Distinct from product-manager (strategy vs delivery)
- **Document Writer** (`skills/document-writer/`, `agents/document-writer.md`), technical writing specialist; API docs, runbooks, onboarding guides, READMEs, ADRs, release notes, Diátaxis framework
- **Kotlin/Java Code Reviewer** (`skills/kotlin-code-reviewer/`, `agents/kotlin-code-reviewer.md`), reviews Kotlin/Java diffs for correctness, idiomatic style, Spring Boot patterns, Flyway migrations, fintech domain logic
- **Frontend Code Reviewer** (`skills/frontend-code-reviewer/`, `agents/frontend-code-reviewer.md`), reviews frontend diffs for React patterns, TypeScript strictness, WCAG accessibility violations, performance regressions
- **SessionStart hook** (`hooks/hooks.json`, `hooks/session-start`), automatically injects orchestrator skill context at every session start; no manual invocation required

### Changed
- `hooks/session-start` updated to reference `skills/orchestrator/SKILL.md`
- All version references bumped to `2.0.0` (`package.json`, `plugin.json`, `marketplace.json`)
- `CONTRIBUTING.md` updated with dual-file requirement (skill + agent) for new roles
- `README.md` rewritten to document orchestrator, sub-agent dispatching, and all 15 specialists organised by category

---

## [1.0.1] - 2026-03-25

### Fixed
- Bumped version in `plugin.json` and `marketplace.json` to `1.0.1` to enable plugin update detection

---

## [1.0.0] - 2026-03-24

### Added
- Claude Code plugin config (`.claude-plugin/plugin.json`, `marketplace.json`)
- 8 role-based skills covering the full SDLC, each grounded in real-world role research:
  - `frontend-designer`, React, CSS, accessibility, Core Web Vitals
  - `backend-engineer`, APIs, databases, system design, security
  - `data-analyst`, SQL, Python/pandas, data storytelling, statistical reasoning
  - `ux-researcher`, qualitative/quantitative methods, NN/g frameworks, journey mapping
  - `senior-engineer`, architecture, code review, mentoring, technical leadership
  - `devex`, CI/CD, DORA metrics, developer tooling, build optimisation
  - `product-manager`, discovery, user stories, prioritisation, outcome-driven roadmaps
  - `qa-engineer`, test strategy, test pyramid, risk-based testing, quality advocacy
- `team` orchestrator skill for automatic role routing and multi-role task sequencing
- All skills include a permission + notification contract (announce → confirm → report)
- `CONTRIBUTING.md` with research-first methodology and skill template
- `.gitignore`, `package.json` (semver), `CHANGELOG.md`
- `CLAUDE.md` with project rules for AI assistants
