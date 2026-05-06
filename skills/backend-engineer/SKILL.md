---
name: backend-engineer
description: Use when designing APIs, working with databases, building microservices, handling authentication and authorisation, optimising server performance, designing data models, or any task involving server-side logic, infrastructure, or system architecture.
version: 2.1.0
---

# Backend Engineer

## Iron Law

```
No new behaviour without a test that fails first. Data outlives code — schema design deserves more care than any single feature.
```

---

## Before Taking Any Action

1. **Announce** what you intend to do and why
2. **Explain the approach** — paradigm, data model, API contract, error handling, trade-offs
3. **Ask for confirmation** before writing any file, running any command, or executing any database query
4. **Report** what was created and flag follow-up items (migrations needed, env vars to set, etc.)

---

## Task Approach

Use this table to determine what to produce for each task type:

| User asks for | What to produce |
|---|---|
| API design | Resource URL structure, HTTP verb mapping, status code table, error response shape (RFC 9457 Problem Details), pagination strategy, versioning decision, and OpenAPI spec outline |
| Data model / schema design | Normalised ER diagram or table definitions, surrogate key choice with rationale, index plan for every query path, migration strategy (forward-only, zero-downtime), and `EXPLAIN ANALYZE` output for non-trivial queries |
| New feature / endpoint implementation | Confirm paradigm (OOP/FP/hybrid), propose API contract and data model first, list reuse candidates found in codebase, then implement with domain logic separated from infrastructure, validation at boundaries, and explicit error handling |
| Code review | Per-concern feedback using severity labels (blocker / major / minor / nit / question / nice); check input validation, credential handling, query indexing, error surfacing, test coverage |
| Authentication / authorisation design | OAuth 2.0 flow selection with rationale, JWT claim validation checklist, token lifetime recommendation, secret storage approach, rate-limit plan for auth endpoints |
| Performance optimisation | Measurement-first: identify bottleneck with profiling data or query plan; propose targeted fix with expected before/after impact; no speculative optimisation |
| Microservice / system design | Service boundary rationale, communication pattern (REST vs gRPC vs events) with trade-offs, data ownership model, failure mode analysis, observability plan |
| Event-driven / messaging design | Delivery guarantee choice (at-least-once / exactly-once) with consumer idempotency requirement, schema evolution strategy (Avro/Protobuf + Schema Registry), dead-letter queue policy |
| Security review | Parameterised queries check, auth boundary audit, secret exposure scan, PII-in-logs check, threat model update |

---

## Paradigm Identification

| Signal | Paradigm | Principles that apply |
|---|---|---|
| Repository/Service/Mapper classes, inheritance hierarchies | OOP | SOLID, GoF design patterns |
| `val`/`const` everywhere, no mutation, pipeline operators | FP | Immutability, pure functions, Railway-Oriented Programming |
| Effect types (`Option`, `Either`, `Result`) in signatures | FP | Algebraic design, typeclass constraints |
| Multi-paradigm language (TS, Python, Kotlin) | Hybrid | SOLID at module boundaries; FP discipline inside function bodies |

---

## SOLID — When to Apply, When NOT To

| Principle | Apply when | Do NOT apply when |
|---|---|---|
| **SRP** | A class changes for two different reasons (different teams, different rates) | Splitting a small, cohesive class — produces shotgun surgery (Fowler, *Refactoring*) |
| **OCP** | Stable behaviour with variant implementations (payment processors, notification channels). Abstract on the third repetition, not the first | Early in the system's life before variation axes are clear |
| **LSP** | Always, when using inheritance or interface implementation — violations must be fixed | N/A |
| **ISP** | Fat interfaces force clients to depend on methods they don't use | Micro-interfaces (one method each) in languages without structural typing — navigation overhead |
| **DIP** | Every boundary you want to test or swap: DB access, external APIs, clock | Value objects, utilities, pure functions — injecting `StringFormatter` is over-engineering |

---

## Most Useful GoF Patterns in Backend Work

| Pattern | Use case | Watch out for |
|---|---|---|
| **Strategy** | Multiple algorithms at runtime: payment processors, pricing rules, discount strategies | Using if/switch on a fixed enum where Strategy adds no value |
| **Factory Method** | Creating objects from runtime context: event deserialisation, DB connection from config | — |
| **Observer / Domain Events** | Notify downstream systems after a write without direct coupling | In-process observer for durable events — use a message broker (Kafka, RabbitMQ) for durability |
| **Decorator** | Cross-cutting concerns layered on a core behaviour: caching, retry, metrics, circuit breaker | — |
| **Adapter** | Wrap third-party SDK behind a domain interface; map errors to domain types | Coupling domain code directly to Stripe/Twilio types |
| **Template Method** | Fixed workflow, variant steps: import pipelines, batch jobs | — |
| **Repository** (DDD) | Abstract data access so domain logic doesn't depend on SQL/ORM specifics | — |

---

## API Design

**REST:**
- URLs are nouns: `/resources/{id}`, `/resources/{id}/sub-resources`
- HTTP verb semantics: GET (safe + idempotent), PUT (idempotent replace), PATCH (partial update), POST (non-idempotent), DELETE (idempotent)
- Status code discipline: 201 Created, 204 No Content, 400 validation, 401 unauthenticated, 403 unauthorized, 404 Not Found, 409 Conflict, 422 semantic failure, 429 rate limit, 503 unavailable
- Pagination: cursor-based (opaque `next_cursor`) for large, frequently-updated datasets; offset only for small stable datasets
- Versioning: URL path (`/v1/`) for breaking changes only — backward-compatible changes don't need a new version
- Error responses: Problem Details (RFC 9457) — `type`, `title`, `status`, `detail`; map all domain exceptions to HTTP status codes in a single global handler

**gRPC:**
- Service-to-service communication; always set a deadline on every RPC call — without deadlines, cascading failures are guaranteed (Nygard, *Release It!*)
- Proto files are contracts — version in a shared repo; never remove a field or change a field number

**Event-driven:**
- Choose REST when you need a synchronous response; choose events when the operation is a fact that happened and multiple systems react to it
- At-least-once delivery (Kafka/SQS default): consumers must be idempotent; use a deduplication key (`event_id`)
- Schema evolution: Avro/Protobuf with a Schema Registry; same backward-compatibility rules as gRPC

**OpenAPI**: document all REST endpoints as a first-class deliverable; treat it as living documentation.

---

## Database Rules

- Start normalised (3NF); denormalise only when a measured read performance problem exists — premature denormalisation creates update anomalies
- Surrogate keys (UUID v7 / ULID) not natural keys — natural keys change; random UUID v4 causes B-tree page splits on insert
- `created_at` and `updated_at` on every table
- Every foreign key column has an index; every unbounded `WHERE` clause on a large table has a covering index; use `EXPLAIN ANALYZE` before shipping any non-trivial query
- Migrations: forward-only, backward-compatible, zero-downtime, reviewed as carefully as application code
- Transaction isolation: READ COMMITTED for standard CRUD; REPEATABLE READ or SERIALIZABLE for read-modify-write financial operations; never hold a DB lock across an external API call (Kleppmann, *Designing Data-Intensive Applications*)
- Optimistic locking (`version` column + reject writes where version ≠ expected) prevents lost updates without DB-level locks

---

## Error Handling

- Wrap all third-party errors at the adapter boundary — `sql.ErrNoRows` becomes `UserNotFoundError` before crossing into the service layer
- OOP: unchecked exceptions for application errors mapped to HTTP status in a single global handler; checked exceptions only for recoverable conditions where the caller must decide
- FP: `Result<T, E>` / `Either<E, A>` for expected failures (validation, not-found); Railway-Oriented Programming (Wlaschin, *Domain Modeling Made Functional*) chains them without nested conditionals
- Include entity ID, operation name, and non-sensitive input values in every error — enough context to diagnose without log-trawling

---

## Security

- Parameterised queries / prepared statements for all SQL — ORM usage does not automatically protect raw query escape hatches
- OAuth 2.0 + Authorization Code + PKCE for user-facing flows; Client Credentials for service-to-service
- JWT: validate `alg`, `iss`, `aud`, `exp`, `iat`, `sub`; short-lived access tokens (15 min) + long-lived refresh tokens (HttpOnly cookie or secure storage)
- Secrets in a vault (AWS Secrets Manager, HashiCorp Vault) or environment variables — never in source code; use `git-secrets` or Gitleaks in CI
- Rate limit input-intensive endpoints (login, signup, password reset) by IP and user fingerprint

---

## Testing

- Unit tests: domain logic, calculations, state machines — isolated, no I/O, mocks only at infrastructure boundaries
- Integration tests: Testcontainers for real PostgreSQL/Redis — never mock the database in integration tests
- Contract tests (Pact): for service-to-service API boundaries in microservices — runs fast without spinning up the other service
- Every public method: happy path + at least one error/edge case
- Reset DB state between tests: `@Transactional` rollback or table truncate; never share mutable state across tests

---

## Observability

- Structured logging (JSON in production) with correlation IDs on every log line; never log PII or credentials
- Metrics for every external call: latency (p99, not average), error rate, throughput
- Distributed tracing (OpenTelemetry) with trace propagation across service boundaries
- Health endpoints (`/health`, `/ready`) on every service
