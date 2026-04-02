# Team of Agents

A Claude Code plugin that gives you a full SDLC team of specialised AI agents — each a distinct expert role — plus an orchestrator that plans tasks and dispatches specialist subagents automatically.

Install once. Invoke any specialist directly, or let the orchestrator plan and coordinate the work for you.

---

## Install

```
/plugin marketplace add pranav8494/team-of-agents
/plugin install team-of-agents@pranav8494
```

---

## The Orchestrator

The orchestrator is the recommended entry point for complex or multi-domain tasks. It:

1. **Understands** the request and identifies which domains are involved
2. **Plans** a work breakdown before doing anything
3. **Announces** the plan and asks for confirmation
4. **Dispatches** specialist subagents — in parallel for independent tasks, sequentially for dependent ones
5. **Synthesises** the results into a unified output

```
/orchestrator   build a fintech dashboard: research user needs, design the UI, and write the API
/orchestrator   review this PR — check the Kotlin backend and the React frontend
/orchestrator   I need to ship a new feature: write requirements, break it into tasks, and create a test plan
```

---

## Specialists

### Engineering

| Skill | Invoke | Best For |
|-------|--------|----------|
| Frontend Designer | `/frontend-designer` | React, CSS, design systems, accessibility, web performance |
| Backend Engineer | `/backend-engineer` | APIs, databases, microservices, authentication, system design |
| Kotlin Backend Engineer | `/kotlin-backend-engineer` | Kotlin/Spring Boot, fintech backend, JVM architecture |
| Fintech Frontend Engineer | `/fintech-frontend-engineer` | React/Tailwind fintech UI, payment flows, Core Web Vitals, SEO |
| Senior Engineer | `/senior-engineer` | Architecture review, technical design, refactoring, trade-off analysis |
| DevEx Engineer | `/devex` | CI/CD pipelines, developer tooling, build optimisation, local dev setup |
| SRE | `/sre` | SLOs/error budgets, production observability, incident response, postmortems, Terraform/Kubernetes, chaos engineering |
| Kotlin/Java Code Reviewer | `/kotlin-code-reviewer` | Review Kotlin/Java diffs, Spring Boot, Flyway migrations |
| Frontend Code Reviewer | `/frontend-code-reviewer` | Review React/TypeScript diffs, accessibility, performance |
| QA Engineer | `/qa-engineer` | Test plans, test cases, automation strategy, quality standards |

### Product

| Skill | Invoke | Best For |
|-------|--------|----------|
| Product Manager | `/product-manager` | Discovery, user stories, roadmaps, prioritisation, success metrics |
| Project Manager | `/project-manager` | Sprint planning, delivery milestones, RAID logs, status reports |
| UX Researcher | `/ux-researcher` | User research, personas, journey maps, usability testing |
| Data Analyst | `/data-analyst` | SQL, data files, metrics, trend analysis, actionable insights |

### GTM

| Skill | Invoke | Best For |
|-------|--------|----------|
| SEO Manager | `/seo-manager` | SEO strategy, keyword research, technical SEO audits, ranking diagnostics |

### Content

| Skill | Invoke | Best For |
|-------|--------|----------|
| Document Writer | `/document-writer` | API docs, runbooks, onboarding guides, READMEs, ADRs, release notes |

---

## Usage Examples

```
# Orchestrator — plans and dispatches automatically
/orchestrator   this feature needs a data model, API, and frontend — coordinate the work
/orchestrator   I'm not sure where to start, here's the brief: [paste brief]

# Engineering
/frontend-designer       create a responsive navbar with a mobile hamburger menu
/backend-engineer        design a REST API for user authentication with JWT
/kotlin-backend-engineer implement a payment idempotency pattern in Spring Boot
/senior-engineer         review this architecture decision for our notification system
/devex                   set up a GitHub Actions CI pipeline for a monorepo
/sre                     define SLOs and error budget policy for our payments API
/sre                     we had a major incident last night — help me write the postmortem
/kotlin-code-reviewer    [paste diff] review this Kotlin service layer change
/frontend-code-reviewer  [paste diff] review this React component for accessibility
/qa-engineer             write a test plan for the login and registration flow

# Product
/product-manager    write user stories for a search feature
/project-manager    create a RAID log and milestone plan for the Q3 release
/ux-researcher      what research methods should I use for onboarding discovery?
/data-analyst       analyse this CSV and find the top trends by region

# GTM & Content
/seo-manager        audit our product pages for technical SEO issues
/document-writer    write a runbook for our deployment process
```

---

## How Agents Behave

Every agent in this team:

1. **Announces** what it intends to do and why before taking any action
2. **Asks for confirmation** before writing files, running commands, or making web requests
3. **Reports** what was done when complete, with clear next steps

No agent will silently modify your codebase. You are always in control.

---

## Sub-agent Dispatching

When invoked via the orchestrator, specialists run as subagents — isolated Claude instances with focused prompts. This means:

- **Parallel execution**: Independent tasks run simultaneously (e.g. backend design + frontend design at the same time)
- **Sequential chaining**: Dependent tasks pass their output as context to the next agent (e.g. UX research findings → product requirements → engineering design)
- **Focused expertise**: Each subagent operates only within its domain and cannot spawn further agents

You can also invoke any specialist directly without going through the orchestrator.

---

## Design Philosophy

Each skill and agent definition in this plugin is grounded in real-world role definitions — not generic AI behaviour with a job title attached. Before writing any skill, the role is researched from reliable industry sources: practitioner writing, job description surveys, academic frameworks, and professional standards bodies.

The goal: each agent behaves like a real practitioner in that field.

---

## Contributing

We welcome new roles and improvements to existing ones. See [CONTRIBUTING.md](./CONTRIBUTING.md).

---

## Repository Structure

```
team-of-agents/
├── .claude-plugin/
│   ├── plugin.json           ← plugin metadata
│   └── marketplace.json      ← marketplace listing
├── agents/                   ← subagent definitions (dispatched by the orchestrator)
│   ├── backend-engineer.md
│   ├── frontend-designer.md
│   ├── kotlin-backend-engineer.md
│   ├── fintech-frontend-engineer.md
│   ├── senior-engineer.md
│   ├── devex.md
│   ├── sre.md
│   ├── kotlin-code-reviewer.md
│   ├── frontend-code-reviewer.md
│   ├── qa-engineer.md
│   ├── product-manager.md
│   ├── project-manager.md
│   ├── ux-researcher.md
│   ├── data-analyst.md
│   ├── seo-manager.md
│   └── document-writer.md
├── hooks/
│   ├── hooks.json            ← SessionStart hook registration
│   └── session-start         ← injects orchestrator skill at session start
├── skills/                   ← user-invocable skill definitions
│   ├── orchestrator/SKILL.md ← plans tasks, dispatches agents, synthesises results
│   ├── backend-engineer/SKILL.md
│   ├── kotlin-backend-engineer/SKILL.md
│   ├── frontend-designer/SKILL.md
│   ├── fintech-frontend-engineer/SKILL.md
│   ├── senior-engineer/SKILL.md
│   ├── devex/SKILL.md
│   ├── sre/SKILL.md
│   ├── kotlin-code-reviewer/SKILL.md
│   ├── frontend-code-reviewer/SKILL.md
│   ├── qa-engineer/SKILL.md
│   ├── product-manager/SKILL.md
│   ├── project-manager/SKILL.md
│   ├── ux-researcher/SKILL.md
│   ├── data-analyst/SKILL.md
│   ├── seo-manager/SKILL.md
│   └── document-writer/SKILL.md
├── CHANGELOG.md
├── CONTRIBUTING.md
└── README.md
```

---

## License

MIT — see [LICENSE](./LICENSE).
