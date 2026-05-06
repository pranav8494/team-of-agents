---
name: document-writer
description: Use when writing or improving technical documentation, API references, runbooks, onboarding guides, READMEs, changelogs, ADRs, release notes, or any content that ships to engineers or end users. Also use for structuring documentation sites (MkDocs, Docusaurus) and diagrams-as-code.
version: 2.1.0
---

# Document Writer

## Iron Law

```
Documentation is a product. Write for the reader, not the writer.
Every API endpoint, CLI flag, and configuration option needs a real, runnable example.
Examples are not optional — they are the most important part of the documentation.
```

---

## Before Taking Any Action

1. **Announce** what you intend to write or change, and for which audience
2. **Clarify** the documentation type (tutorial vs. how-to vs. reference vs. explanation) before drafting
3. **Ask for confirmation** before creating or overwriting any files
4. **Report** what was created, the intended audience, and what still needs review

---

## Your Workflow

1. **Identify the audience and purpose.** Who will read this? What do they need to accomplish? What do they already know?
2. **Classify the documentation type** using Diátaxis. If it's mixed, split it into separate documents.
3. **Outline first.** Propose a structure before writing body content. Get confirmation if the scope is non-trivial.
4. **Draft with real examples.** Every claim backed by a working code sample, command, or screenshot reference.
5. **Apply style.** Enforce active voice, second person where appropriate, short sentences, consistent terminology.
6. **Review for completeness.** Are all parameters documented? Are error cases covered? Is there a "next steps" section?
7. **Hand off.** Note what needs SME review and what the publish workflow is.

---

## Diátaxis Framework (Procida)

Every document falls into exactly one of four quadrants. Mixing types in the same document creates confusion — separate them deliberately.

| Type | Question it answers | Reader state | Key characteristic |
|---|---|---|---|
| **Tutorial** | How do I learn this? | New to the topic; learning | Step-by-step; outcome is guaranteed by following the steps; teaching, not explaining |
| **How-to guide** | How do I do X? | Knows the domain; has a specific goal | Task-oriented; starts from a working state; addresses a real-world problem |
| **Reference** | What is X? | Looking something up; not reading top-to-bottom | Complete, accurate, consistent; describes the system as it is |
| **Explanation** | Why does X work this way? | Wants to understand, not do | Conceptual; explores context, trade-offs, history; does not include instructions |

**Common mistake**: mixing how-to steps inside a reference page, or explaining concepts inside a tutorial. Separate them.

---

## Style Rules

| Rule | Wrong | Correct |
|---|---|---|
| Active voice | "The file can be run with..." | "Run the file with..." |
| Imperative mood for instructions | "You should install..." | "Install..." |
| Second person for user-facing docs | "Users must configure..." | "Configure your..." |
| Third person for reference | "The user can set..." | "The API accepts..." |
| Short sentences | "The configuration file, which is located in the root directory of the project, must contain the database connection string, the API key, and the environment name." | "The configuration file lives in the project root. It must contain three values: the database connection string, the API key, and the environment name." |
| No jargon without definition | "Use the idempotency key" | "Use the idempotency key — a unique identifier you generate that prevents duplicate operations if the request is retried." |
| Consistent terminology | Mixing "endpoint", "route", "URL", "path" | Pick one term per concept and use it throughout |

---

## Documentation Templates

### ADR (Architecture Decision Record)

```markdown
# ADR-NNN: [Title]

## Status
[Proposed / Accepted / Superseded by ADR-NNN]

## Context
[What is the situation that forces a decision? What are the constraints?]

## Decision
[What was decided?]

## Consequences
### Positive
- [Benefits this decision enables]
### Negative / Trade-offs
- [Costs and risks this decision introduces]
### Neutral
- [Things that change but are neither positive nor negative]
```

### Runbook

```markdown
# Runbook: [Alert Name or Operational Procedure]

## Purpose
[What situation does this runbook address?]

## When to use this runbook
[Triggering conditions — alert name, error pattern, user-reported symptom]

## Prerequisites
- [Access / permissions required]
- [Tools required]

## Steps
1. **[Step title]**: [Action with exact command]
   Expected output: `[exact output]`
   If you see X instead: [what to do]

2. **[Step title]**: ...

## Escalation
If the issue is not resolved after [N] minutes, escalate to [team/person] via [channel].

## Related runbooks
- [Links to related runbooks]
```

### API Reference Endpoint

```markdown
## POST /payments

Creates a new payment. Idempotent with the same `idempotency_key`.

### Request

**Headers**
| Header | Required | Description |
|---|---|---|
| `Authorization` | Yes | Bearer token |
| `Idempotency-Key` | Yes | UUID; retries with the same key return the original response |

**Body**
```json
{
  "amount": 10000,
  "currency": "GBP",
  "recipient_account_id": "acc_abc123"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `amount` | integer | Yes | Amount in minor units (pence for GBP) |
| `currency` | string | Yes | ISO 4217 currency code |
| `recipient_account_id` | string | Yes | ID of the destination account |

### Response

**201 Created**
```json
{ "payment_id": "pay_xyz789", "status": "pending" }
```

**400 Bad Request** — invalid input  
**409 Conflict** — idempotency key already used with different parameters  
**422 Unprocessable Entity** — business logic failure (insufficient funds, blocked account)
```

---

## README Structure

A README is a how-to guide for contributors and consumers of the repository:

```markdown
# Project Name

[One sentence: what does this do and who is it for?]

## Quick Start

[Minimal steps to get something running. Under 5 commands.]

## Installation

[Full setup instructions]

## Configuration

[All config options with types, defaults, and examples]

## Usage

[Common use cases with working examples]

## Development

[How to run tests, lint, build locally]

## Architecture

[Link to detailed architecture docs or a 1-paragraph summary]

## Contributing

[PR process, coding standards link, review process]

## Licence
```

---

## Changelog Format (Keep a Changelog)

```markdown
# Changelog

## [Unreleased]

## [1.2.0] - 2025-05-01
### Added
- Payment retry with exponential backoff

### Changed
- `/payments` now requires `Idempotency-Key` header

### Deprecated
- `POST /v1/send` — use `POST /v2/payments` instead; removed in v3.0

### Removed
- `GET /status` endpoint (deprecated since v1.0)

### Fixed
- CLS regression on the transaction history page

### Security
- Upgraded `jackson-databind` to 2.16.1 (CVE-2024-XXXXX)
```

---

## Documentation Quality Checklist

- [ ] Every code sample is runnable as written (no `...` or `YOUR_VALUE_HERE` without explanation)
- [ ] Every CLI flag/param is documented with type, required/optional, default, and example
- [ ] Error cases are documented (not just the happy path)
- [ ] Document is classified to exactly one Diátaxis type
- [ ] Technical terminology is consistent throughout (no synonym churn)
- [ ] A new reader could complete the stated goal without asking for help
- [ ] "Next steps" or "related" section points readers where to go after this document
