# Contributing to team-of-agents

Thank you for contributing. This project aims to be a best-practices reference for role-based AI agents grounded in real-world role definitions.

---

## The Research-First Rule

> **Before writing or updating any `SKILL.md` or agent definition, research the role from reliable external sources.**

The goal is for each agent to behave like a *real* practitioner in that field, not a generic AI assistant with a job title attached.

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
version: 2.1.0
---

# Role Title

## Iron Law

[2-3 non-negotiable principles that govern every task this skill handles]

---

## Before Taking Any Action

1. Announce what you intend to produce and why
2. Confirm scope with the user
3. Ask for confirmation before writing files, running commands, or accessing external resources
4. Report what was produced when complete

---

## Task Approach

| User asks for | What to produce |
|---|---|
| [Task type] | [Concrete output description] |

---

## [Role-Specific Knowledge Sections]

[Decision tables, named frameworks, checklists grounded in real practice]

---

## Output Protocol

End every response with:
CONFIDENCE: [High|Medium|Low], [one-line reason]

If out of scope or missing context:
BLOCKED: [reason], [what would unblock this]
```

### 3. Create the agent definition (subagent for orchestrator dispatch)

```bash
touch agents/your-role-name.md
```

Agent definitions are condensed system prompts used when the orchestrator dispatches this specialist as a subagent. They share the same Iron Law and Task Approach table as the corresponding skill, but are shorter and optimised for focused, isolated execution:

```markdown
---
name: your-role-name
description: "[One-line description of what this agent handles. Returns X.]"
model: inherit
tools: Read, Write, Edit, Bash, Glob, Grep
disallowedTools: Agent
---

# Role Title

## Iron Law

[Same Iron Law as the skill]

## Task Approach

| User asks for | What to produce |
|---|---|
| [Task type] | [Concrete output description] |

## Expertise

- [Key domain knowledge, tools, frameworks]

## Output Format

- For implementation: working code with inline comments on non-obvious decisions
- For design: concise proposal with trade-off notes
- For analysis: structured findings with specific, actionable recommendations
- For review: per-item feedback with severity label; overall verdict

End every response with:
CONFIDENCE: [High|Medium|Low], [one-line reason]

If out of scope or missing context:
BLOCKED: [reason], [what would unblock this]
```

**Important:** Always include `disallowedTools: Agent`, subagents cannot spawn further agents.

### 4. Update the orchestrator routing table

Add the new role to `skills/orchestrator/SKILL.md` in the appropriate category (Engineering, Product, GTM, or Content) with a one-line description of what it's best for.

### 5. Test your skill

Invoke your skill directly in Claude Code:
- [ ] The Iron Law is present and contains 2-3 non-negotiable principles
- [ ] The Task Approach table is present and covers the role's core task types
- [ ] The Output Protocol is present and emits a CONFIDENCE or BLOCKED signal at the end of every response
- [ ] It announces intended actions before taking them
- [ ] It asks for confirmation before writing files or running commands
- [ ] Role-specific knowledge sections (decision tables, checklists) are grounded in real practice

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

## Adding a Domain or Stack Overlay

The frontend skills (`frontend-planner`, `frontend-engineer`, `frontend-reviewer`) are
domain-agnostic. Domain knowledge (fintech, travel, healthcare) and stack knowledge (a framework and
its versions) live in `overlays/` and are inherited, never copied into a forked skill.

**To add one:** copy `overlays/domains/_TEMPLATE.md` or `overlays/stacks/_TEMPLATE.md` and fill in the
three required headings. Keep them verbatim, the base skills look for them by name:

```markdown
## Plan deltas     ← read by frontend-planner
## Build deltas    ← read by frontend-engineer
## Review deltas   ← read by frontend-reviewer
```

**Rules:**
- **Deltas only.** A rule that applies to any domain belongs in the base skill. An overlay that
  restates base rules has started drifting, delete the restatement.
- **Stack overlays are version-stamped** and carry a verify-before-applying header. The installed
  versions in the target repo always win over the overlay.
- **Review deltas carry severity labels** (`[blocker]`, `[major]`, `[minor]`) so they slot straight
  into a review.
- **Research-First applies here too.** Cite sources in the PR, especially for stack overlays.

**Optional shim.** To make `/travel-frontend-engineer` resolve, add a short
`skills/travel-frontend-engineer/SKILL.md` that invokes `frontend-engineer` and applies the overlay.
Use `skills/fintech-frontend-engineer/SKILL.md` as the model. A shim contains no rules of its own.

**Test it.** Invoke the shim and confirm the response announces both parts, e.g.
"frontend-engineer + travel overlay". A silently skipped overlay is the failure mode this mechanism
has to guard against, so the announcement is the thing to verify.

---

## Improving an Existing Role

If you believe an existing skill misrepresents the role:

1. Open an issue describing what is inaccurate and why
2. Include sources that support a better representation
3. Submit a PR with specific changes and your reasoning

Changes to existing skills will be reviewed carefully, they affect everyone using the plugin.

---

## Versioning

This project follows [Semantic Versioning](https://semver.org/):

- **Patch** (`x.x.1`), bug fixes to existing skills (typos, minor clarifications)
- **Minor** (`x.1.0`), new roles added
- **Major** (`1.0.0`), breaking changes to skill names, plugin structure, or orchestrator behaviour

Update `CHANGELOG.md`, `package.json`, `.claude-plugin/plugin.json`, and `.claude-plugin/marketplace.json` in your PR.

---

## Code of Conduct

Be kind. Disagreements about role definitions should be resolved with evidence and sources, not opinions.
