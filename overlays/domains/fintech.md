# Fintech overlay

Money, regulated disclosure, and sensitive data. Deltas only — general React, accessibility, and
testing rules stay in the base skills.

## Plan deltas

- **Currency and locale are decided before tasks are written**, per money value. A task that says
  "show the balance" without naming both is not ready.
- **Precision is a planning decision:** amounts arrive from the backend as strings or integer minor
  units. If a task implies arithmetic in the browser, the plan is wrong — move it server-side.
- **Mutating money calls need an idempotency key strategy** named in the plan, not discovered later.
- **Every screen is classified public or authenticated.** Authenticated screens are `noindex` and
  excluded from the sitemap; account data must never sit at a crawlable URL.
- **Disclosure text (APR, risk warnings, regulatory references) is flagged for legal review** and never
  drafted by the implementer.
- Conventions file gains: currency/locale policy · money representation · masking rules · retention and
  logging rules for PII.

## Build deltas

| Wrong | Right |
|---|---|
| `"£" + amount.toFixed(2)` | `Intl.NumberFormat(locale, { style: 'currency', currency })` |
| `0.1 + 0.2` on money | Integer minor units, or a decimal library — never a float |
| `"1.2M"` hardcoded | `Intl.NumberFormat` with `notation: 'compact'` |
| `"3.5%"` hardcoded | `Intl.NumberFormat` with `style: 'percent'` |
| Blank or hidden zero | Explicit `£0.00` — blank amounts generate support calls |
| `-£50.00` unlabelled | Signed and colour-coded, sign from `Intl`, never colour alone |
| `"2 hours ago"` on an audit record | Exact datetime via `Intl.DateTimeFormat` with explicit timezone |

- **Masking is the default state**, revealed only on explicit user action: card `•••• •••• •••• 4242`,
  sort code `••-••-42`, account `••••1234`. Mask before the value enters JSX, never in CSS.
- **Never** `console.log` card data, account numbers, or PII — including in development.
- **Never** put PII in a URL. IDs only.
- **Never** auto-copy a sensitive value to the clipboard.
- Irreversible actions (payment, withdrawal, transfer) require an explicit review step before submit,
  and a disabled submit control while in flight.
- Completed transactions are immutable in the UI. Corrections are new records with visible history.
- Test fixtures use synthetic data. Real PII never enters a repo, including in test files.

## Review deltas

- `[blocker]` Money arithmetic in JavaScript numbers.
- `[blocker]` Card number, account number, or PII logged, put in a URL, or rendered unmasked by default.
- `[blocker]` Authenticated screen without `noindex`, or account data reachable at a crawlable URL.
- `[blocker]` Retryable money mutation with no idempotency key.
- `[major]` Amount rendered without explicit currency and locale.
- `[major]` Zero or negative amount unhandled, or sign conveyed by colour alone.
- `[major]` Irreversible action with no confirmation step, or submit not disabled in flight.
- `[major]` Relative timestamp on an audit-critical record.
- `[minor]` Formatting utility without tests for zero, negative, very large, and a second locale.
