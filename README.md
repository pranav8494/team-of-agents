<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=6,7,8&height=220&section=header&text=TEAM%20OF%20AGENTS&fontSize=60&fontColor=fff&animation=fadeIn&fontAlignY=40&desc=Full%20SDLC%20team%20for%20Claude%20Code&descSize=20&descAlignY=65&descAlign=50" width="100%" alt="Team of Agents" />

Hire 17 domain experts in one install. Invoke any specialist directly,<br>
or let the orchestrator plan, dispatch, and synthesise the work for you.

[![Specialists](https://img.shields.io/badge/specialists-17-6366f1?style=flat-square)](./skills)
[![Domains](https://img.shields.io/badge/domains-engineering%20%2B%20product%20%2B%20GTM-10b981?style=flat-square)](.)
[![Execution](https://img.shields.io/badge/execution-parallel-f59e0b?style=flat-square)](.)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-plugin-cc785c?style=flat-square)](.)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](./LICENSE)

[Install](#install) · [How It Works](#how-it-works) · [Orchestrator](#the-orchestrator) · [Specialists](#specialists) · [Examples](#usage-examples) · [Contributing](./CONTRIBUTING.md) · [Changelog](./CHANGELOG.md)

</div>

---

## See it in action

**One request. Multiple specialists dispatched in parallel. One synthesised answer.**

<table>
<tr>
<td width="33%">

**Understand + Plan**

Orchestrator reads your request, identifies domains, and breaks work into subtasks, each assigned to the right specialist with a trust tier.

</td>
<td width="33%">

**Dispatch in parallel**

Independent tasks run simultaneously. Dependent tasks chain findings forward. Key facts are shared across rounds via a findings scratchpad.

</td>
<td width="34%">

**Synthesise**

Confidence signals are checked, conflicts resolved by a senior-engineer review pass, and everything assembled into one coherent answer with a full trace log.

</td>
</tr>
</table>

```
/orchestrator   my checkout conversion is dropping, help me figure out why and fix it

→ Identified: UX Researcher + Data Analyst + Frontend Reviewer  [parallel]
→ UX Researcher    [High]  users don't trust the payment form
→ Data Analyst     [High]  mobile abandonment is 2× desktop
→ Frontend Reviewer [High] submit button is below the fold on mobile

→ Chaining findings → Backend Engineer + Frontend Designer  [parallel]
→ Senior Engineer review pass...
→ Synthesis complete. Full trace log attached.
```

---

## What it does

- **Orchestrates multi-role tasks automatically**, drop in a brief and the orchestrator decomposes it, assigns specialists, runs them in parallel where possible, and assembles one answer
- **17 specialists across the full SDLC**, backend, frontend, Kotlin, SRE, DevEx, QA, product, project, UX, data, SEO, document writer, and more
- **Parallel execution built-in**, independent tasks run simultaneously; sequential tasks chain findings forward automatically
- **Confidence signals on every output**, every specialist returns `High / Medium / Low` so the orchestrator and you know what needs a second look
- **No silent changes**, every specialist announces what it will do, waits for your approval before writing files or running commands, and reports what was done
- **Trust tiers**, T1 (research), T2 (file writes), T3 (commands), declared in the plan so you always know what requires your sign-off

---

## Install

```
/plugin marketplace add pranav8494/team-of-agents
/plugin install team-of-agents@team-of-agents
```

---

## How It Works

Think of this plugin as a consultant firm. The orchestrator is the project manager, it listens, breaks the problem into pieces, and hands each piece to the right expert. Each expert does their slice and hands the work back. The project manager assembles it into one answer.

### The three layers

**You** describe a task in plain English.

**The orchestrator** reads your request, identifies which specialists are needed, shows you the plan, waits for approval, dispatches each specialist, collects their answers, and produces one combined result.

**The specialists** (17 of them) each handle one domain, backend, frontend, data, UX, SRE, and so on. Each does its slice and nothing more.

### What happens step by step

```
You type a request
       ↓
Orchestrator reads it → breaks it into subtasks → shows you the plan
       ↓
You confirm → orchestrator dispatches specialists
(independent tasks run in parallel; dependent tasks run sequentially)
       ↓
Each specialist returns output + a confidence signal (High / Medium / Low)
       ↓
Orchestrator checks confidence:
  High   → use it directly
  Medium → flag the assumption, ask you to confirm before acting
  Low    → surface the gap, re-dispatch with more context
  BLOCKED → ask you what's missing, then retry
       ↓
Optional: senior-engineer reviews all outputs for errors or conflicts
       ↓
Orchestrator assembles everything into one answer with a full trace log
```

---

## The Orchestrator

The recommended entry point for complex or multi-domain tasks.

```
/orchestrator   build a fintech dashboard: research user needs, design the UI, and write the API
/orchestrator   review this PR, check the Kotlin backend and the React frontend
/orchestrator   I need to ship a new feature: write requirements, break it into tasks, create a test plan
```

**Phases:**

1. **Understand**, reads the request, identifies domains and dependencies
2. **Plan**, breaks work into subtasks, assigns a specialist and trust tier to each
3. **Announce**, shows you the plan and waits for your approval
4. **Dispatch**, sends each specialist a structured context envelope; parallel tasks run simultaneously
5. **Synthesise**, checks confidence signals, assembles an agent trace log, produces a unified result
6. **Critic pass** *(optional)*, senior-engineer reviews everything and flags errors, conflicts, and gaps

**Trust tiers**, declared upfront so you know what requires your sign-off:

| Tier | Type | Confirmation |
|------|------|-------------|
| T1 | Research and analysis | Used directly, no confirmation needed |
| T2 | Files written to disk | Shown to you first, you confirm before saving |
| T3 | Commands and infrastructure | Explicit approval required for each action |

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
| SRE | `/sre` | SLO/error budget design, observability, runbooks, postmortems, IaC |
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
| Technical Business Analyst | `/technical-business-analyst` | Implementation plans, requirements, bridging business goals to engineering specs |

### GTM & Content

| Skill | Invoke | Best For |
|-------|--------|----------|
| SEO Manager | `/seo-manager` | SEO strategy, keyword research, technical SEO audits, ranking diagnostics |
| Document Writer | `/document-writer` | API docs, runbooks, onboarding guides, READMEs, ADRs, release notes |

---

## Usage Examples

```
# Orchestrator, plans and dispatches automatically
/orchestrator   this feature needs a data model, API, and frontend, coordinate the work
/orchestrator   I'm not sure where to start, here's the brief: [paste brief]

# Engineering
/frontend-designer       create a responsive navbar with a mobile hamburger menu
/backend-engineer        design a REST API for user authentication with JWT
/kotlin-backend-engineer implement a payment idempotency pattern in Spring Boot
/senior-engineer         review this architecture decision for our notification system
/devex                   set up a GitHub Actions CI pipeline for a monorepo
/sre                     define SLOs and error budget policy for our payments API
/sre                     we had a major incident last night, help me write the postmortem
/kotlin-code-reviewer    [paste diff] review this Kotlin service layer change
/frontend-code-reviewer  [paste diff] review this React component for accessibility
/qa-engineer             write a test plan for the login and registration flow

# Product
/product-manager             write user stories for a search feature
/project-manager             create a RAID log and milestone plan for the Q3 release
/ux-researcher               what research methods should I use for onboarding discovery?
/data-analyst                analyse this CSV and find the top trends by region
/technical-business-analyst  write an implementation plan for the payments migration

# GTM & Content
/seo-manager      audit our product pages for technical SEO issues
/document-writer  write a runbook for our deployment process
```

---

## How Agents Behave

Every specialist in this team:

1. **Announces** what it intends to produce and why before taking any action
2. **Asks for confirmation** before writing files, running commands, or making web requests
3. **Signals confidence**, `CONFIDENCE: High / Medium / Low`, so the orchestrator and you know how much to trust the output
4. **Signals when blocked**, returns `BLOCKED: [reason], [what would unblock it]` rather than guessing

No agent will silently modify your codebase. You are always in control.

---

## Sub-agent Dispatching

When invoked via the orchestrator, specialists run as subagents, isolated Claude instances with focused prompts:

- **Parallel execution**, independent tasks run simultaneously
- **Sequential chaining**, dependent tasks pass output as context to the next agent
- **Shared findings scratchpad**, key facts from parallel agents are passed to the next wave
- **Focused expertise**, each subagent operates only within its domain
- **Partial failure recovery**, if one agent fails, synthesis continues with remaining outputs rather than stalling the whole run

---

## Design Philosophy

Skills in this plugin are task guides, not personas. Each skill defines what Claude should *produce* for a given task type, not who Claude should *be*. The reference frameworks, decision tables, and checklists are tools Claude reaches for during specific tasks, not background knowledge for a character.

Each skill is grounded in real-world practice: DORA, SPACE, Diátaxis, SOLID, Fowler's Technical Debt Quadrant, Google SRE Book, Nielsen Norman Group, and others. The goal is that each agent behaves like a real practitioner in that field, not a generic AI with a job title attached.

---

## Repository Structure

```
team-of-agents/
├── .claude-plugin/
│   ├── plugin.json           ← plugin metadata + skills manifest
│   └── marketplace.json      ← marketplace listing
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
│   ├── document-writer/SKILL.md
│   └── technical-business-analyst/SKILL.md
├── hooks/
│   ├── hooks.json            ← SessionStart hook registration
│   └── session-start         ← injects orchestrator skill at session start
├── CHANGELOG.md
├── CONTRIBUTING.md
└── README.md
```

---

## Contributing

We welcome new roles and improvements to existing ones. See [CONTRIBUTING.md](./CONTRIBUTING.md).

---

## License

MIT, see [LICENSE](./LICENSE).