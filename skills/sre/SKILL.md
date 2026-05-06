---
name: sre
description: Use when defining or auditing SLOs/SLIs/error budgets, designing production observability (metrics, alerting, dashboards), reviewing production readiness, reducing operational toil, writing runbooks or postmortems, designing infrastructure as code, or planning chaos experiments. Distinct from devex — this skill covers production reliability and the outer loop (deploy → monitor → incident → postmortem), not developer productivity tools or CI/CD pipeline speed.
version: 2.1.0
---

# Site Reliability Engineering

## Iron Law

```
70% of outages are caused by changes — make every change incremental, observable, and reversible.
Alert on symptoms, not causes — every page that does not require immediate human action is a bug.
MTTR beats MTBF — optimise for fast recovery, not for preventing all failures.
```

---

## Before Taking Any Action

1. **Announce** what you intend to produce — SLO proposal, alert rules, runbook, IaC, postmortem, PRR report, chaos experiment design
2. **Confirm scope** — which service, which environment, existing SLOs if any
3. **Ask for confirmation** before writing files, running commands, or proposing infrastructure changes
4. **Report** what was produced, what decisions were embedded, and what still needs human review (SLO targets need stakeholder sign-off; chaos experiments need a maintenance window)

---

## Task Approach

Use this table to determine what to produce for each task type:

| User asks for | What to produce |
|---|---|
| SLO / SLI definition | SLI selection per service type (see framework below) + SLO target + error budget calculation + error budget policy table |
| Observability design | Four Golden Signals instrument checklist + Prometheus metrics + Grafana dashboard JSON or dashboard-as-code spec + alert rules |
| Alert rules | Burn rate alerts (fast + slow) per SLO using the two-alert model; include runbook link placeholders |
| Production readiness review | PRR report structured against the six-dimension checklist; flag each dimension as Pass / At Risk / Fail with remediation steps |
| Runbook | Step-by-step response procedure for a specific alert or failure mode; include detection, diagnosis steps, mitigation commands, escalation path, and rollback |
| Postmortem | Completed postmortem document using the blameless template; ask for the incident timeline and impact before writing |
| Toil identification | Toil audit: list recurring manual tasks, classify each against the toil criteria, propose automation approach and elimination priority |
| Infrastructure as code | Terraform modules or Kubernetes manifests following the IaC principles below; always include rollback plan as a comment block |
| Chaos experiment design | Hypothesis statement + blast radius definition + success/failure criteria + kill switch + measurement plan; do not generate commands that inject failure without explicit user confirmation |

---

## SLO/SLI/Error Budget Framework

### Definitions

| Term | Definition |
|---|---|
| **SLI** | A quantitative measure of service behaviour (e.g. % of requests with latency < 200ms) |
| **SLO** | A target for the SLI over a time window (e.g. 99.9% of requests < 200ms over 30 days) |
| **Error Budget** | `1 − SLO target` — the allowed failure rate (e.g. 0.1% of requests may fail the SLI) |
| **SLA** | An SLO with contractual consequences (penalties, credits) |

### SLI Selection by Service Type

| Service type | Recommended SLIs |
|---|---|
| Request/response (APIs) | % of requests with latency < threshold (p99); % of successful responses (non-5xx) |
| Batch / pipeline | Throughput (records/s); end-to-end latency; % of batches completing within SLA |
| Storage | Durability (data loss rate); read latency p99; write availability |
| Scheduled jobs | % of jobs completing within time budget; error rate |

**Use percentiles (p99, p95), never averages.** An average hides the tail — users on p99 latency are real users having a bad experience.

### Error Budget Calculation

```
Error Budget = (1 − SLO%) × Time Window
Example: 99.9% SLO over 30 days
Error Budget = 0.1% × 30 × 24 × 60 = 43.2 minutes
```

### Error Budget Policy

| Budget remaining | Action |
|---|---|
| > 50% | Full velocity; deploy freely |
| 25–50% | Normal operations; monitor burn rate |
| 10–25% | Elevated caution; no risky deployments without SRE review |
| < 10% | Reliability sprint; feature freeze; postmortem required for any incident |
| 0% exhausted | Feature freeze until budget resets; escalate to engineering leadership |

---

## Four Golden Signals

Build dashboards starting with these four. If all four are healthy, the service is probably healthy.

| Signal | What it measures | Alert condition |
|---|---|---|
| **Latency** | Response time for successful and failed requests | p99 > SLO threshold for > 5 minutes |
| **Traffic** | Request rate (RPS, events/s, QPS) | Anomalous spike or drop from baseline |
| **Errors** | Rate of failed requests (4xx client, 5xx server) | Error rate > SLO-derived burn rate |
| **Saturation** | CPU, memory, disk, connection pool utilisation | > 80% sustained for > 10 minutes |

Alert on symptoms (p99 latency exceeds SLO threshold), not causes (CPU > 80%). Causes are investigated; symptoms are paged.

---

## Alert Design Rules

- **Every page requires immediate human action.** A page that does not is a bug — it trains engineers to ignore alerts.
- **Alert on burn rate, not absolute values.** A 1-hour burn rate of 14.4× will exhaust a 30-day 99.9% budget in 5 days.
- **Two-alert model per SLO:**
  - Fast burn: 6-hour window, 5% budget consumed → page immediately
  - Slow burn: 3-day window, 10% budget consumed → ticket/warning
- **Alert fatigue is a failure mode.** Any alert firing > 5 times per week without action is a toil candidate.
- **Runbook link in every alert.** Engineers under pressure must not have to remember what to do.

---

## Production Readiness Review (PRR) Checklist

Run before a service goes to production or receives on-call coverage:

| Dimension | Pass criteria |
|---|---|
| **System architecture** | Dependencies documented; failure modes identified; no unmitigated single points of failure |
| **SLI instrumentation** | Four Golden Signals instrumented; dashboards deployed; alerts configured and tested |
| **Emergency response** | Runbook written and reviewed; incident response flow practiced by at least one engineer |
| **Capacity planning** | Load tested to 120% of expected peak; N+2 provisioned |
| **Change management** | Canary or blue/green configured; rollback tested and documented; deployment < 30 minutes |
| **Performance targets** | SLOs defined, agreed with product team, and baselined against load test results |

A service that fails PRR is not blocked from deployment — it is blocked from receiving on-call coverage. The engineering team retains on-call responsibility until PRR passes.

---

## Postmortem Template (Blameless)

```markdown
## Incident Summary
**Date**: [Date]
**Duration**: [Start time → End time]
**Severity**: [P0 / P1 / P2]

## Impact
- Users affected: [number / segment]
- Features affected: [what stopped working]
- Revenue / SLA impact: [if applicable]

## Timeline (all times UTC)
| Time | Event |
|---|---|
| HH:MM | [Alert fired / mitigation attempted / resolved] |

## Root Cause
[Avoid "human error" as a root cause — it is always a system design issue.
Describe the systemic conditions that allowed the incident to occur.]

## Contributing Factors
- [Factor 1]
- [Factor 2]

## What Went Well
- [Things that helped contain or resolve the incident]

## What Went Poorly
- [Things that made the incident worse or harder to resolve]

## Action Items
| Action | Owner | Due Date | Status |
|---|---|---|---|
| [Specific improvement] | [Name] | [Date] | Open |
```

**Blameless principle:** a person made a decision that made sense given the information and tools available at the time. The system allowed that decision to cause harm. Fix the system.

---

## Toil Identification

**Toil** (Google SRE Book): work that is manual, repetitive, automatable, interrupt-driven, and produces no enduring value beyond the immediate action.

| Toil candidate | Automation approach |
|---|---|
| Manual credential rotation | Automated rotation on schedule + secrets manager integration |
| Restarting a service when it OOMs | HPA autoscaling + memory limits + alert on degraded pod |
| Recurring alert → same fix every time | Runbook → automate the fix → eliminate the root cause |
| Manual scaling before known traffic events | Scheduled scaling policy |
| Answering "what's the status of X?" | Status page + automated incident status updates |

**Hard rule:** if toil exceeds 50% of available engineering time, the team loses capacity for systemic improvements. Prioritise elimination of the highest-frequency items first.

---

## Infrastructure as Code Principles

- All infrastructure changes are version-controlled, reviewed, and applied through CI/CD — never via console or SSH
- Terraform: one state file per environment; remote state with locking (S3 + DynamoDB or Terraform Cloud)
- Kubernetes: resource requests and limits on every container; `PodDisruptionBudget` for every stateful workload
- Every infrastructure change has a rollback plan documented before it is applied to production
- Secrets are never in Terraform state — use Vault or AWS Secrets Manager with references, not values

---

## Chaos Engineering Principles

When designing a chaos experiment, always produce these four elements before any execution:

1. **Hypothesis** — "The service will continue to serve requests at SLO if [failure condition]"
2. **Blast radius** — scope to a test environment or a small percentage of traffic; define the kill switch
3. **Measurement plan** — which SLIs to observe; what constitutes pass vs fail
4. **Rollback trigger** — the condition at which the experiment is aborted immediately

Do not generate commands that inject failure into production without explicit user confirmation and a defined maintenance window.

---

## Output Protocol

End every response with a confidence signal on its own line:

```
CONFIDENCE: [High|Medium|Low] — [one-line reason]
```

- **High** — output is complete, correct, and based on sufficient context
- **Medium** — output is reasonable but contains an assumption or a gap; state the assumption inline
- **Low** — insufficient context to produce a reliable result; state what is missing

If the task is outside this skill's scope or you lack the information needed to proceed, return this instead of a confidence signal:

```
BLOCKED: [reason] — [what information would unblock this]
```

Do not guess or produce low-quality output to avoid returning BLOCKED. A precise BLOCKED is more useful than a low-confidence guess.
