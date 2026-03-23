# Contributing to agents-team

Thank you for contributing. This project aims to be a best-practices reference for role-based AI agents grounded in real-world role definitions.

---

## The Research-First Rule

> **Before writing or updating any `SKILL.md`, research the role from reliable external sources.**

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

### 1. Research the role

Before writing a single line of the skill, research:
- What does this role actually do day to day?
- What skills and tools do practitioners in this role use? (check surveys and job descriptions)
- What separates a great practitioner from an average one?
- How do they think and communicate with other roles?

### 2. Create the skill file

```bash
mkdir skills/your-role-name
touch skills/your-role-name/SKILL.md
```

### 3. Follow the skill template

Every skill must include:

```markdown
---
name: your-role-name
description: "One-line trigger description. Be specific about when to invoke this role."
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

### 4. Test your skill

Invoke your skill in Claude Code and verify:
- [ ] The agent adopts the correct persona and expertise
- [ ] It announces intended actions before taking them
- [ ] It asks for confirmation before writing files or running commands
- [ ] The expertise listed matches what the role actually does in practice

### 5. Submit a PR

Your PR description should include:
- The role you're adding and why it belongs in a software development team
- The sources you used for your research
- A brief summary of what makes this skill accurate to the real role

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

- **Patch** (`1.0.x`) — bug fixes to existing skills (typos, minor clarifications)
- **Minor** (`1.x.0`) — new roles added
- **Major** (`x.0.0`) — breaking changes to skill structure or plugin format

Update `CHANGELOG.md` and `package.json` version in your PR.

---

## Code of Conduct

Be kind. Disagreements about role definitions should be resolved with evidence and sources, not opinions.
