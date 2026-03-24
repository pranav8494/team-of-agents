# agents-team

A Claude Code plugin that gives you a team of specialized AI agents — each a distinct expert role covering the full software development lifecycle.

Install once. Invoke any specialist directly from your Claude Code session.

---

## Install

```
/plugin marketplace add pranav8494/agents-team 
/plugin install agents-team@pranav8494
```

---

## The Team

| Skill | Invoke | Best For |
|-------|--------|----------|
| Frontend Designer | `/frontend-designer` | UI components, CSS, React, design systems, accessibility, web performance |
| Backend Engineer | `/backend-engineer` | APIs, databases, microservices, authentication, system design |
| Data Analyst | `/data-analyst` | Data files, SQL, metrics, charts, trends, insights |
| UX Researcher | `/ux-researcher` | User research, personas, journey maps, usability testing |
| Senior Engineer | `/senior-engineer` | Architecture, code review, technical design, refactoring |
| DevEx Engineer | `/devex` | CI/CD pipelines, developer tooling, local dev setup, DX improvements |
| Product Manager | `/product-manager` | User stories, requirements, roadmaps, prioritisation |
| QA Engineer | `/qa-engineer` | Test plans, test cases, automation strategy, quality standards |
| Team (auto-route) | `/team` | Not sure which role? Let the orchestrator decide |

---

## Usage Examples

```
/frontend-designer  create a responsive navbar with a mobile hamburger menu
/backend-engineer   design a REST API for user authentication with JWT
/data-analyst       analyse data.csv and find the top trends by region
/ux-researcher      what research methods should I use for onboarding discovery?
/senior-engineer    review this function for security and performance issues
/devex              set up a GitHub Actions CI pipeline for a Node.js project
/product-manager    write user stories for a search feature
/qa-engineer        write a test plan for the login and registration flow
/team               I need help improving my app's performance
```

---

## How Each Agent Behaves

Every agent in this team:

1. **Announces** what it wants to do and why before taking any action
2. **Asks for confirmation** before writing files, running commands, or making web requests
3. **Reports** what was done when complete, with clear next steps

No agent will silently modify your codebase. You are always in control.

---

## Design Philosophy

Each skill in this plugin is grounded in real-world role definitions — not generic AI behaviour with a job title attached. Before writing any skill, we research the role from reliable industry sources: practitioner blogs, job description surveys, academic frameworks, and professional standards bodies.

The goal: each agent should behave like a real practitioner in that field.

---

## Contributing

We welcome new roles and improvements to existing ones.

**Adding a new role** is straightforward — see [CONTRIBUTING.md](./CONTRIBUTING.md).

**Improving an existing role** — if you have domain expertise and think a skill misrepresents the role, open an issue or submit a PR with your reasoning and sources.

---

## Repository Structure

```
agents-team/
├── .claude-plugin/
│   ├── plugin.json          ← plugin metadata
│   └── marketplace.json     ← marketplace listing
├── skills/
│   ├── frontend-designer/SKILL.md
│   ├── backend-engineer/SKILL.md
│   ├── data-analyst/SKILL.md
│   ├── ux-researcher/SKILL.md
│   ├── senior-engineer/SKILL.md
│   ├── devex/SKILL.md
│   ├── product-manager/SKILL.md
│   ├── qa-engineer/SKILL.md
│   └── team/SKILL.md        ← orchestrator
├── CHANGELOG.md
├── CONTRIBUTING.md
└── README.md
```

---

## License

MIT — see [LICENSE](./LICENSE).
