# team-of-agents — Implementation Plan

## What We're Building

A **Claude Code plugin** that gives users a team of specialized AI agents — each a distinct expert role covering the full SDLC. Users install it once and invoke roles directly in Claude Code. No Python, no server, no separate terminal.

```
/plugin install team-of-agents@pranav8494
```

Then use it like:
```
/frontend-designer  design a login page with dark mode
/backend-engineer   design a REST API for user authentication
/data-analyst       analyze sales.csv and find trends
/ux-researcher      research onboarding best practices
/senior-engineer    review this PR for architecture issues
/devex              set up CI/CD for this project
/product-manager    write user stories for the checkout flow
/qa-engineer        write a test plan for the login feature
/team               I need help with my app  ← auto-routes
```

---

## Distribution Model (Same as superpowers)

- **Format:** GitHub repo with `.claude-plugin/` config + `skills/` markdown files
- **Install:** `/plugin install team-of-agents@pranav8494` in Claude Code
- **Update:** `/plugin update team-of-agents`
- **Contribute:** Add a new `skills/role-name/SKILL.md` + PR

No npm, no pip, no MCP server needed.

---

## Roles (8 + orchestrator — full SDLC coverage)

| Skill | Covers |
|-------|--------|
| `frontend-designer` | UI, components, CSS, React, design systems, accessibility |
| `backend-engineer` | APIs, databases, microservices, system design |
| `data-analyst` | Data, CSV, SQL, charts, metrics, insights |
| `ux-researcher` | User flows, research, personas, usability testing |
| `senior-engineer` | Architecture, code review, refactoring, mentoring |
| `devex` | CI/CD, tooling, developer productivity, DORA metrics |
| `product-manager` | Requirements, roadmap, user stories, prioritization |
| `qa-engineer` | Test strategy, test plans, quality gates, edge cases |
| `team` | Orchestrator — auto-routes to the right role(s) |

---

## Research-First Rule (Applies to Every Role)

> Before writing any `SKILL.md`, research the role from reliable external sources.
> The goal: the agent should behave like a *real* practitioner, not a generic AI with a job title.

Sources per role:
- Job descriptions from top tech companies (Google, Stripe, Shopify, etc.)
- Industry surveys (Stack Overflow, State of JS/CSS, Kaggle)
- Nielsen Norman Group (UX), DAMA (data), SPACE/DORA frameworks (DevEx)
- Practitioner blogs (Smashing Magazine, Martin Fowler, Google Engineering)

This rule also belongs in `CONTRIBUTING.md` so future contributors follow it.

---

## Project Structure

```
team-of-agents/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── skills/
│   ├── frontend-designer/SKILL.md
│   ├── backend-engineer/SKILL.md
│   ├── data-analyst/SKILL.md
│   ├── ux-researcher/SKILL.md
│   ├── senior-engineer/SKILL.md
│   ├── devex/SKILL.md
│   ├── product-manager/SKILL.md
│   ├── qa-engineer/SKILL.md
│   └── team/SKILL.md
├── CHANGELOG.md
├── CONTRIBUTING.md
├── LICENSE
├── README.md
├── package.json
└── .gitignore
```

---

## Implementation Stories

Each story = one focused session + one git commit.

| # | Story | Done When |
|---|-------|-----------|
| **S1** | Repo Setup | `.gitignore`, `package.json` (v1.0.0), `CHANGELOG.md`, folder structure created and committed |
| **S2** | Plugin Config | `.claude-plugin/plugin.json` + `marketplace.json` created and committed |
| **S3** | Frontend Designer Skill | Web research done → `skills/frontend-designer/SKILL.md` written and committed |
| **S4** | Backend Engineer Skill | Web research done → `skills/backend-engineer/SKILL.md` written and committed |
| **S5** | Data Analyst Skill | Web research done → `skills/data-analyst/SKILL.md` written and committed |
| **S6** | UX Researcher Skill | Web research done → `skills/ux-researcher/SKILL.md` written and committed |
| **S7** | Senior Engineer Skill | Web research done → `skills/senior-engineer/SKILL.md` written and committed |
| **S8** | DevEx Engineer Skill | Web research done → `skills/devex/SKILL.md` written and committed |
| **S9** | Product Manager Skill | Web research done → `skills/product-manager/SKILL.md` written and committed |
| **S10** | QA Engineer Skill | Web research done → `skills/qa-engineer/SKILL.md` written and committed |
| **S11** | Team Orchestrator Skill | `skills/team/SKILL.md` written and committed |
| **S12** | README | Install instructions, all roles documented, usage examples, committed |
| **S13** | CONTRIBUTING.md | Research-first rule, role template, PR guidelines, committed |
| **S14** | Tag + Push | All committed, tagged `v1.0.0`, pushed to GitHub |

---

## Skill File Pattern (Template)

```markdown
---
name: role-name
description: "One-line trigger description for when Claude should invoke this skill."
---

# Role Title

## Who You Are
[Persona — seniority, years of experience, specialization]

## Your Expertise
[Bullet list of technical skills, tools, frameworks — grounded in research]

## How You Think
[Mental models, priorities, decision-making approach]

## How You Communicate
[Tone, style, what you produce, how you interact with other roles]

## Before Taking Any Action
Always:
1. State what you want to do and why
2. Ask for user confirmation before writing/editing files, running commands, or web requests
3. Report what was done after completing

## Your Workflow
[Step-by-step approach to a typical task in this role]
```

---

## Verification (After S14)

```bash
/plugin install team-of-agents@pranav8494

/frontend-designer  create a navbar component in React
/backend-engineer   design a REST API for user auth
/data-analyst       what patterns are in data.csv?
/ux-researcher      how should I design an onboarding flow?
/senior-engineer    review this function for issues
/devex              add a GitHub Actions workflow for tests
/product-manager    write user stories for search feature
/qa-engineer        write a test plan for the login page
/team               I need help improving my app's performance
```
