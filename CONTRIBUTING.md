# Contributing to team-of-agents

Thank you for contributing. This project aims to be a best-practices reference for role-based AI agents grounded in real-world role definitions.

---

## The Research-First Rule

> **Before writing or updating any `SKILL.md` or agent definition, research the role from reliable external sources.**

The goal is for each agent to behave like a *real* practitioner in that field — not a generic AI assistant with a job title attached.

**What counts as a reliable source:**
- Job descriptions from well-known tech companies (Google, Stripe, Shopify, etc.)
- Industry surveys (Stack Overflow Developer Survey, State of JS/CSS, Kaggle)
- Professional standards bodies (Nielsen Norman Group for UX, DAMA for data)
- Academic or industry frameworks (DORA/SPACE for DevEx, WCAG for accessibility)
- Respected practitioner writing (Martin Fowler, staffeng.com, SVPG, Smashing Magazine)

**What does not count:**
- Your own assumptions about the role
- GPT/LLM-generated summaries without primary sources
- A single job description from one company

Include your sources in the PR description.

---

## Adding a New Role

Every new role requires **two files**: a user-invocable skill and a subagent definition.

### 1. Research the role

Before writing a single line, research:
- What does this role actually do day to day?
- What skills and tools do practitioners use? (check surveys and job descriptions)
- What separates a great practitioner from an average one?
- How do they think and communicate with other roles?

### 2. Create the skill file (user-invocable)

```bash
mkdir skills/your-role-name
touch skills/your-role-name/SKILL.md
```

Every skill must follow this template:

```markdown
---
name: your-role-name
description: "Use when [specific trigger condition]. One sentence, be specific."
---

# Role Title

## Who You Are
[Persona — seniority, years of experience, core identity, what they optimise for]

## Your Expertise
[Technical skills, tools, frameworks — all grounded in your research]

## How You Think
[Mental models, decision-making approach, priorities, known biases to counteract]

## How You Communicate
[Tone, output format, how they interact with other roles on the team]

## Before Taking Any Action
Always:
1. State what you want to do and why
2. Ask for user confirmation before writing/editing files, running commands, or making web requests
3. Report what was done after completing

## Your Workflow
[Step-by-step approach to a typical task in this role]
```

### 3. Create the agent definition (subagent for orchestrator dispatch)

```bash
touch agents/your-role-name.md
```

Agent definitions are condensed system prompts used when the orchestrator dispatches this specialist as a subagent. They are shorter than skills and optimised for focused, isolated execution:

```markdown
---
name: your-role-name
description: "[Specialist type]. Invoke for [specific tasks]. Returns [output format]."
model: inherit
tools: Read, Write, Edit, Bash, Glob, Grep
disallowedTools: Agent
---

# Role Title

[2-3 sentence persona. Core expertise areas. Philosophy in one sentence.]

## Expertise
[Bullet list of specific skills, tools, frameworks]

## How You Work
[5-7 numbered steps — concise workflow for this role's core tasks]

## Output Format
[What the agent returns: implementation, analysis, structured artefact, etc.]
```

**Important:** Always include `disallowedTools: Agent` — subagents cannot spawn further agents.

### 4. Update the orchestrator routing table

Add the new role to `skills/orchestrator/SKILL.md` in the appropriate category (Engineering, Product, GTM, or Content) with a one-line description of what it's best for.

### 5. Test your skill

Invoke your skill directly in Claude Code:
- [ ] The agent adopts the correct persona and expertise
- [ ] It announces intended actions before taking them
- [ ] It asks for confirmation before writing files or running commands
- [ ] The expertise listed matches what the role actually does in practice

Test via the orchestrator:
- [ ] The orchestrator correctly routes tasks to your new role
- [ ] The agent definition works when dispatched as a subagent

### 6. Submit a PR

Your PR description should include:
- The role you're adding and why it belongs in the team
- The sources you used for your research
- A brief summary of what makes this skill accurate to the real role
- Confirmation that both `skills/` and `agents/` files were created

---

## Improving an Existing Role

If you believe an existing skill misrepresents the role:

1. Open an issue describing what is inaccurate and why
2. Include sources that support a better representation
3. Submit a PR with specific changes and your reasoning

Changes to existing skills will be reviewed carefully — they affect everyone using the plugin.

---

## Versioning

This project follows [Semantic Versioning](https://semver.org/):

- **Patch** (`x.x.1`) — bug fixes to existing skills (typos, minor clarifications)
- **Minor** (`x.1.0`) — new roles added
- **Major** (`1.0.0`) — breaking changes to skill names, plugin structure, or orchestrator behaviour

Update `CHANGELOG.md`, `package.json`, `.claude-plugin/plugin.json`, and `.claude-plugin/marketplace.json` in your PR.

---

## Code of Conduct

Be kind. Disagreements about role definitions should be resolved with evidence and sources, not opinions.
