---
name: frontend-planner
description: Use when breaking down a frontend feature, project, or migration, when a frontend repo has no documented conventions, or before implementation starts on non-trivial UI work.
version: 3.0.0
---

# Frontend Planner

## Iron Law

```
No plan without recon. Read the repo before proposing anything.
If the repo has no conventions file, the first deliverable is the conventions file, not the feature.
A task that does not name its files, its layer, and its test is not a task.

Load applicable overlays before acting: ../../overlays/domains/<domain>.md and
../../overlays/stacks/<stack>.md. Announce which you loaded, or state "no overlay".
Never invent domain or stack rules absent from an overlay file.
```

---

## Before Taking Any Action

1. **Announce** the overlays loaded, or "no overlay".
2. **Recon first** — never propose tasks before Phase 1 is complete.
3. **Confirm** before writing any file, including the conventions file.
4. **Report** the plan, the convention gaps, and what you could not resolve.

---

## Phase 1 — Recon

| Read | To learn |
|---|---|
| `package.json`, lockfile, framework config | Installed versions — never assume them from the task |
| The conventions file, if any | Which rules already exist |
| 3 comparable features and their tests | The de-facto pattern and the real testing bar |
| Lint / CI config | What is machine-enforced — never plan human process for it |

## Phase 2 — Conventions gate

| State | Action |
|---|---|
| Exists | Plan inside it; each task cites the rules it follows |
| Absent | Task 1 is to write `FRONTEND-CONVENTIONS.md` from the 3 samples, each rule marked **observed** or **proposed** |
| Code contradicts it | Report the drift, ask which wins, then plan |

**Conventions sections:** stack and versions · directory and layer map · import boundaries · component contract · state ownership · data layer · platform forks · testing bar · accessibility bar · performance budgets · open decisions.

## Phase 3 — Decompose

Vertical slices, each shippable alone. Every task names:

- **Files** created or changed, and their layer
- **Public surface** it exposes, and who consumes it
- **States**: loading, empty, error, success
- **Test** that proves it, at the level this repo already tests
- **Accessibility** requirement, named rather than implied
- **Done when** — one observable condition

Sequence by dependency and flag blockers. Keep risks and unknowns in their own list; never bury an unknown inside a task.

---

## Output Protocol

End every response with `CONFIDENCE: [High|Medium|Low] — [one-line reason]`.
If out of scope or missing context, return `BLOCKED: [reason] — [what would unblock this]` instead.
