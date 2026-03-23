# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

---

## [1.0.0] - 2026-03-24

### Added
- Claude Code plugin config (`.claude-plugin/plugin.json`, `marketplace.json`)
- 8 role-based skills covering the full SDLC, each grounded in real-world role research:
  - `frontend-designer` — React, CSS, accessibility, Core Web Vitals
  - `backend-engineer` — APIs, databases, system design, security
  - `data-analyst` — SQL, Python/pandas, data storytelling, statistical reasoning
  - `ux-researcher` — qualitative/quantitative methods, NN/g frameworks, journey mapping
  - `senior-engineer` — architecture, code review, mentoring, technical leadership
  - `devex` — CI/CD, DORA metrics, developer tooling, build optimisation
  - `product-manager` — discovery, user stories, prioritisation, outcome-driven roadmaps
  - `qa-engineer` — test strategy, test pyramid, risk-based testing, quality advocacy
- `team` orchestrator skill for automatic role routing and multi-role task sequencing
- All skills include a permission + notification contract (announce → confirm → report)
- `CONTRIBUTING.md` with research-first methodology and skill template
- `.gitignore`, `package.json` (semver), `CHANGELOG.md`
- `CLAUDE.md` with project rules for AI assistants
