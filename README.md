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

## How It Works

Think of this plugin as hiring a consultant firm. When you bring them a problem, a **project manager** listens, breaks the problem into pieces, and hands each piece to the right expert — the architect, the designer, the data person. Each expert does their slice, hands back their work, and the project manager assembles it into a single answer for you.

That's exactly what this plugin does, but the "people" are AI skills and the "project manager" is the **orchestrator**.

### The three layers

**You** describe a task in plain English.

**The orchestrator** reads your request, figures out which specialists are needed, shows you its plan, waits for your approval, sends each specialist their assignment, collects their answers, and hands you one combined result.

**The specialists** (17 of them) each handle one domain — backend, frontend, data, UX, SRE, and so on. Each one does its slice of the work and nothing more.

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

### A concrete example

You ask: *"My checkout conversion is dropping — help me figure out why and fix it."*

1. Orchestrator splits it into: UX research + data analysis + frontend review
2. All three run in parallel
3. UX researcher finds: users don't trust the payment form
4. Data analyst finds: mobile abandonment is 2× desktop
5. Frontend reviewer finds: the submit button is below the fold on mobile
6. Orchestrator passes those findings to: backend engineer (add trust signals) + frontend designer (fix the mobile layout)
7. Senior engineer reviews all outputs for conflicts
8. You get one coherent plan with every specialist's work woven together

---

## The Orchestrator

The orchestrator is the recommended entry point for complex or multi-domain tasks.

```
/orchestrator   build a fintech dashboard: research user needs, design the UI, and write the API
/orchestrator   review this PR — check the Kotlin backend and the React frontend
/orchestrator   I need to ship a new feature: write requirements, break it into tasks, and create a test plan
```

**Phases:**

1. **Understand** — reads the request, identifies domains and dependencies
2. **Plan** — breaks work into subtasks, assigns a specialist and trust tier to each
3. **Announce** — shows you the plan and waits for your approval
4. **Dispatch** — sends each specialist a structured context envelope (task, background, constraints, output format); parallel tasks run simultaneously
5. **Synthesise** — checks confidence signals, assembles an agent trace log, and produces a unified result
6. **Critic pass** *(optional)* — if any output is uncertain or conflicting, senior-engineer reviews everything and flags errors, conflicts, assumptions, and gaps before you see the final answer

**Trust tiers** — declared in the plan so you always know what requires your sign-off:
- **T1** Research and analysis — used directly, no confirmation needed
- **T2** Files written to disk — shown to you first, you confirm before saving
- **T3** Commands and infrastructure — explicit approval required for each action

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
| SRE | `/sre` | SLO/error budget design, observability, runbooks, postmortems, IaC, chaos experiments |
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
3. **Signals confidence** at the end of every response — `CONFIDENCE: High / Medium / Low` — so the orchestrator (and you) know how much to trust the output
4. **Signals when blocked** — if a specialist lacks the context to proceed, it returns `BLOCKED: [reason] — [what would unblock it]` rather than guessing
5. **Reports** what was done when complete, with clear next steps

No agent will silently modify your codebase. You are always in control.

---

## Sub-agent Dispatching

When invoked via the orchestrator, specialists run as subagents — isolated Claude instances with focused prompts. This means:

- **Parallel execution** — independent tasks run simultaneously (e.g. backend design + frontend design at the same time)
- **Sequential chaining** — dependent tasks pass their output as context to the next agent (e.g. UX research findings → product requirements → engineering design)
- **Shared findings scratchpad** — key facts from parallel agents are captured and passed to the next wave of agents, so nothing is lost between rounds
- **Focused expertise** — each subagent operates only within its domain
- **Partial failure recovery** — if one agent fails or is blocked, the orchestrator retries with a narrower scope; if that also fails, synthesis continues with the remaining outputs rather than stalling the whole run

---

## Design Philosophy

Skills in this plugin are task guides, not personas. Each skill defines what Claude should *produce* for a given task type — not who Claude should *be*. The reference frameworks, decision tables, and checklists in each skill are tools Claude reaches for during specific tasks, not background knowledge for a character.

Each skill is grounded in real-world practice: practitioner writing, job description surveys, academic frameworks (DORA, SPACE, Diátaxis, SOLID, Fowler's Technical Debt Quadrant, Google SRE Book, Nielsen Norman Group, and others). The goal is that each agent behaves like a real practitioner in that field, not a generic AI with a job title attached.

---

## Contributing

We welcome new roles and improvements to existing ones. See [CONTRIBUTING.md](./CONTRIBUTING.md).

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

## License

MIT — see [LICENSE](./LICENSE).
