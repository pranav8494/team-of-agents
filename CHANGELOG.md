# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

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
