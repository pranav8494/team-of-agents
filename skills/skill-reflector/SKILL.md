---
name: skill-reflector
description: Use when reviewing session learnings, approving or discarding captured patterns, promoting recurring patterns into permanent skill improvements, or proposing PRs to the plugin repo. Invoke after any session where the team-of-agents captured new learnings.
version: 1.0.0
---

# Skill Reflector

## Iron Law

```
A learning that is never reviewed is noise. A learning reviewed and approved becomes
institutional memory. The only purpose of this skill is to close that loop.
```

---

## Before Taking Any Action

1. Announce the project root you resolved and the log file you will read.
2. Present all unreviewed entries as a numbered review table — never approve anything silently.
3. Wait for explicit user input before writing to any approved-learnings.json.
4. Report final counts: N approved, N skipped, and any PR candidates surfaced.

---

## Phase 1 — Read Pending Entries

Resolve the project root with this one Bash command, the same rule the orchestrator and the
session-start hook use:

```bash
DIR="${CLAUDE_PROJECT_DIR:-$PWD}"; git -C "$DIR" rev-parse --show-toplevel 2>/dev/null || echo "$DIR"
```

Then read the session log with the Read tool (use Read, not bash, for every file in this skill):

- `{project root}/.team-of-agents/session-log.jsonl`

The orchestrator captures every learning here. Scope (project or global) is decided at approval time,
not capture time, so there is no global session log.

Filter to entries where `reviewed: false`. If the file does not exist or has no unreviewed entries,
say so and skip to Phase 4.

---

## Phase 2 — Present Review Table

Render all unreviewed entries as a single numbered table.

```
#  | Skill             | Category   | Summary
---|-------------------|------------|------------------------------------
1  | backend-engineer  | preference | Use snake_case for DB column names
2  | frontend-engineer | correction | Avoid inline styles in React components
3  | qa-engineer       | pattern    | Always test the empty-list edge case
```

Then ask: "Enter numbers to approve (e.g. 1 3), 'all', or 'none'."

For the approved entries, ask for scope in one question:

"Scope for each? `project` applies only in this repo, `global` applies in every project.
Default is project. (e.g. `1 project, 3 global`, or `all project`)"

---

## Phase 3 — Process Approvals

For each approved entry:

1. Read the target approved-learnings.json for the chosen scope:
   - project: `{project root}/.team-of-agents/approved-learnings.json`
   - global: `~/.claude/plugins/team-of-agents/approved-learnings.json`
   If it does not exist, treat it as `[]` and create its directory before writing.
2. Check whether an existing entry for the same skill already says the same thing. Summaries are
   written fresh each session, so match on meaning, not exact text, and name the match you found.
   - If yes: increment `seen_count` by 1, leave all other fields unchanged.
   - If no: append a new entry with `seen_count: 1`, `pr_proposed: false`, and `scope` set to the chosen scope.
3. Show the exact diff to the user and ask for confirmation before writing.
4. Write changes using the Edit or Write tool.
5. Mark the original session-log entry `reviewed: true`.

For each skipped entry, mark `reviewed: true` without writing to approved-learnings.json.

Mark entries reviewed with the Edit tool, changing `"reviewed":false` to `"reviewed":true` on that
entry's line only. Never rewrite the whole log with Write, since another session may have appended to it.

---

## Phase 4 — PR Threshold Check

After processing, read both approved-learnings.json files (project and global, whichever exist) and
check for entries where:

- `seen_count >= 3` AND
- `pr_proposed == false`

For each match, surface this block:

```
[Skill Reflector] Recurring pattern detected — seen 3+ times:
  Skill:    {skill}
  Summary:  {summary}
  Detail:   {detail}

Propose a PR to the plugin repo to make this permanent? (yes / no)
```

If the user answers yes:

1. Show the exact text to be added to the plugin's `skills/{skill}/SKILL.md` — a minimal addition
   (one bullet, one table row, or one rule block). Do not rewrite the whole file.
2. The change belongs in the plugin repo (`pranav8494/team-of-agents`), not the project you are in, and
   the installed plugin copy is not a git checkout. Ask: "Path to your local clone of team-of-agents?
   (or `none` to just get the text)"
3. If the user gives a path, confirm it is a clone of the plugin repo
   (`git -C <path> remote get-url origin` mentions `team-of-agents`). Then show the commands and ask
   "Run these? (yes / no)" before running any of them:
   ```bash
   cd <path>
   git switch -c learning/{skill}-<short-slug>
   # apply the edit to <path>/skills/{skill}/SKILL.md with the Edit tool, using the absolute path
   git commit -am "skill({skill}): <short title>"
   git push -u origin HEAD
   gh pr create --repo pranav8494/team-of-agents --title "skill({skill}): <short title>" \
     --body "Recurring pattern approved {seen_count} times via skill-reflector."
   ```
   Write `<short title>` yourself from the summary, without quotes or backticks.
4. If the user answers `none`, or the path is not a clone of the plugin repo, print the proposed text
   and the link `https://github.com/pranav8494/team-of-agents/issues/new` so they can file it. Do not
   edit files in the current project.
5. Set `pr_proposed: true` on the entry in the file it came from.

If the user answers no, set `pr_proposed: true` as well so the same pattern is not proposed every run.

---

## Phase 5 — Report

```
Skill Reflector complete.
  Approved:             N entries written to approved-learnings.json
  Skipped:              N entries marked reviewed, not approved
  PR candidates surfaced: N
```

---

## File Schemas

**session-log.jsonl** — one JSON object per line:
```json
{"id":"...","session_id":"...","skill":"...","category":"preference|correction|pattern","summary":"...","detail":"...","captured_at":"...","reviewed":false}
```

**approved-learnings.json** — JSON array:
```json
[{"id":"...","skill":"...","category":"...","summary":"...","detail":"...","approved_at":"...","seen_count":1,"pr_proposed":false,"scope":"project|global"}]
```

---

## Output Protocol

End every response with a confidence signal on its own line:

```
CONFIDENCE: [High|Medium|Low] — [one-line reason]
```

- **High** — output is complete, correct, and based on sufficient context
- **Medium** — output is reasonable but contains an assumption or a gap; state the assumption inline
- **Low** — insufficient context to produce a reliable result; state what is missing

If the task is outside this skill's scope or you lack the information needed to proceed, return this instead:

```
BLOCKED: [reason] — [what information would unblock this]
```
