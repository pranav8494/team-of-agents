---
name: kotlin-dev
description: Use when implementing any feature or bugfix in a Kotlin/Spring Boot service — covers layer conventions, idioms, testing, and observability
version: 2.1.0
---

# Kotlin Developer Skill

## Iron Law

```
No new behaviour without a test that fails first, then passes.
```

---

## Before Taking Any Action

1. **Announce** what you intend to do and why
2. **Explain the approach** — data model decisions, Spring configuration, security implications, migration steps, trade-offs
3. **Ask for confirmation** before writing or editing any file, running any command, or executing any database operation
4. **Report** what was created or changed, and flag follow-up items (new env vars, Flyway migrations to run, Spring Security config to update)

---

## Task Approach

Use this table to determine what to produce for each task type:

| User asks for | What to produce |
|---|---|
| New feature / endpoint | Clarify idempotency, consistency, and compliance requirements; propose data model and API contract first; search codebase for reuse candidates; implement thin controller → service → adapter/repository with validation at the controller boundary and domain exceptions mapped at the adapter boundary |
| Bug fix | Reproduce with a failing test first; identify whether the fault is in controller, service, adapter, or data layer; fix at the root cause and confirm the test passes |
| Data model / schema | Normalised table definitions, surrogate key choice, index plan for every query path, Flyway migration (forward-only, backward-compatible, zero-downtime), `EXPLAIN ANALYZE` for non-trivial queries |
| Code review | Per-layer feedback: constructor injection, resilience wrapping on external calls, `@Transactional` scope, `BigDecimal` for currency, idempotency of financial operations, PII/card data absent from logs, Flyway migration safety, index coverage, metrics on new external calls |
| API design | RESTful resource structure, HTTP status code table, Problem Details error shape (RFC 9457), OpenAPI (Springdoc) spec, Bean Validation placement |
| Testing | Unit test with `@WebMvcTest` + MockK/Mockito-Kotlin for controllers and services; Testcontainers integration test for repositories; WireMock for outbound HTTP; property-based tests for financial edge cases |
| Observability / metrics | Micrometer counter/timer with `{org}.{domain}.{action}` naming, `result` tag (success/failure), OkHttp metrics listener registration, `KotlinLogging` lambda form with correlation and entity IDs |
| Performance optimisation | Identify bottleneck with `EXPLAIN ANALYZE` or profiling; tune HikariCP pool size; introduce coroutine parallelism (`async`/`awaitAll`) for independent I/O; cache stable reads with `@Cacheable` |
| Security configuration | Spring Security JWT/OAuth2 resource server config in dedicated `SecurityConfig`, `@PreAuthorize` placement, secret storage guidance, fintech compliance checklist |

---

## Architecture Decision Review

Before designing or implementing anything non-trivial, identify which architectural decisions are in play. For each that applies, follow this pattern:

1. **Name the decision** — what needs to be chosen and why it matters here
2. **Present at least 3 options** — with pros, cons, and the conditions under which each is the right choice
3. **Recommend one** — state which you recommend for this specific context and why
4. **Ask the user to confirm or choose** — do not proceed until the key decisions are confirmed

Common decisions to look for (apply only those relevant to the task):

| Decision area | Examples of options to present |
|---|---|
| **Caching strategy** | (1) No cache — simplest, always fresh; (2) `@Cacheable` with TTL — reduces load, tolerates staleness; (3) `@CacheEvict` on write (event-driven invalidation) — fresh on write, more complex; (4) Write-through — always consistent, write overhead. Recommend based on read/write ratio and freshness requirements. |
| **Consistency model** | (1) Strong consistency — `SERIALIZABLE` transactions, required for financial writes; (2) Eventual consistency — async propagation, suits feeds/analytics; (3) Read-your-writes — middle ground for user-facing writes. Recommend based on data criticality. |
| **Communication pattern** | (1) Synchronous REST/gRPC — simple, immediate response, tight coupling; (2) Async messaging (Kafka/SQS) — decoupled, durable, adds operational complexity; (3) Hybrid — sync for commands needing a result, async for side-effects. Recommend based on latency requirements and coupling tolerance. |
| **External API integration** | (1) Call on every request — simplest, always fresh, may be costly or rate-limited; (2) Cache with TTL — reduces calls, introduces staleness; (3) Background sync + local store — most resilient, adds sync complexity. Use resilience4j circuit breaker + backoff regardless of choice. |
| **Data ownership** | (1) Own the data locally — fast reads, sync burden; (2) Fetch from source service at runtime — always fresh, adds latency and coupling; (3) CQRS read model — optimised reads, eventual consistency. Recommend based on read frequency and staleness tolerance. |
| **Scalability approach** | (1) Vertical scaling — simple, has a ceiling; (2) Horizontal scaling with stateless design — flexible, requires externalised state; (3) Queue-based load levelling — smooths bursts, adds async complexity. Recommend based on bottleneck type (read/write/compute). |

Not every decision applies to every task. Identify the ones that do, present the options, make a recommendation, and confirm with the user before writing code.

---

## Fintech Rules (Non-Negotiable)

- Currency: always `BigDecimal`, never `Double` or `Float`
- Every financial operation must be idempotent — enforce at the API and database layer
- Audit trails are immutable — never update, only append
- Never log, store, or transmit payment card data or PII in plaintext
- Transaction isolation: use `SERIALIZABLE` for financial writes; understand the implications before defaulting to `READ_COMMITTED`
- `@Transactional` scope must not span external API calls — hold DB locks for DB work only

---

## Layer Conventions

### Controller
- `@RestController` + `@RequestMapping`
- **Primary constructor injection only** — never `@Autowired`
- Default `@RequestParam` values inline at the parameter
- `@ResponseStatus(HttpStatus.NO_CONTENT)` on delete endpoints
- No business logic — delegate entirely to service layer

### Service
- `@Service` annotation
- `@Cacheable` / `@CacheEvict` for cached reads of stable data
- Concurrent I/O with coroutines:
  ```kotlin
  runBlocking(Dispatchers.IO) {
      items.map { async { fetch(it) } }.awaitAll()
  }
  ```
- Fire-and-forget async: `CoroutineScope(Dispatchers.IO).launch { try { ... } catch (e: Exception) { logger.error(e) { "..." } } }`
- Wrap all external calls through a resilience executor (circuit breaker / retry)

### Adapter / Client
- `@Repository` or `@Component` depending on role
- HTTP clients: register metrics event listener (e.g. OkHttp `OkHttpMetricsEventListener`)
- gRPC: use coroutine stubs, bridge to sync with `runBlocking`
- **Map HTTP errors to domain exceptions at the adapter boundary**, never in service or controller:
  - 404 → `NotFoundException`
  - 429 → `TooManyRequestsException`
  - 403 → `ForbiddenException`

### Repository
- Aggregates multiple clients; wraps all calls with the resilience executor
- Use Spring Data JPA for standard CRUD; drop to JDBC or native SQL for complex queries

### Domain Models
- `data class` for all DTOs and domain objects
- Nullable fields with `?` for optional attributes
- Collection fields default to `emptyList()`
- No validation annotations on DTOs — validate at the controller boundary

### Exceptions
Extend a base `HttpException` with the appropriate `HttpStatus`:
```kotlin
class NotFoundException(override val message: String) :
    HttpException(status = HttpStatus.NOT_FOUND, message = message)
```
- All domain exceptions live in one package (e.g. `data/model/exception/`)
- Use sealed result types for expected failure paths (insufficient funds, duplicate request); reserve exceptions for truly unexpected conditions

### Mappers
- `object` with extension functions on receiver types — not a Spring bean:
  ```kotlin
  object DomainMapper {
      fun ContentfulDto.toDomain(): DomainModel = ...
  }
  ```
- Return `null` when required data is absent; use `mapNotNull` at call sites

### Configuration
- `@ConfigurationProperties(prefix = "...")` on a `data class` with constructor defaults
- Spring Security: configure JWT / OAuth2 resource server in a dedicated `SecurityConfig`; use `@PreAuthorize` for method-level access control

---

## Code Reuse & Simplicity

- Search the codebase for existing services, utilities, and Spring beans before writing new code
- Prefer extension functions and utility objects over inheritance hierarchies
- Prefer Spring's built-in abstractions (exception handlers, converters, validators) over custom frameworks
- Use sealed class hierarchies for domain result types — avoid raw exceptions for expected outcomes
- Keep controllers thin: delegate to services; keep services focused on one concern

---

## Required Kotlin Idioms

| Situation | Use |
|---|---|
| Null guard + use | `x?.let { use(it) }` |
| Null fallback | `x ?: default` |
| Transform + filter | `mapNotNull`, `filter`, `map` |
| Index by key | `associateBy { it.id }` |
| Group | `groupBy { it.type }` |
| Side effect on value | `also { log(it) }` |
| Enum with string ID | `enum class X(val id: String)` |
| Domain result type | `sealed class Result<out T>` — not nullable returns |

---

## PostgreSQL

- Schema design: normalise balanced against query performance; every unbounded query path needs a covering index
- Indexing: B-tree, partial, composite, covering indexes; use `EXPLAIN ANALYZE` before shipping any new query
- ACID transactions; advisory locks for distributed coordination
- Row-level security for multi-tenant data isolation
- Flyway: version-controlled, forward-only, backward-compatible, zero-downtime migrations
- Connection pooling: tune HikariCP (`maximumPoolSize`, `connectionTimeout`) and monitor pool metrics

---

## API Design

- RESTful resource design: idempotent operations, correct HTTP status codes, explicit versioning strategy
- Error responses: Problem Details (RFC 9457) — `type`, `title`, `status`, `detail`
- OpenAPI (Springdoc): document all endpoints; treat as a first-class deliverable
- Input validation: Bean Validation at the controller boundary; fail fast before any business logic runs

---

## Testing

### Unit Tests (controllers, services, pure logic)
```kotlin
@ExtendWith(SpringExtension::class)
@WebMvcTest(controllers = [MyController::class])
class MyControllerTest {
    @Autowired private lateinit var mockMvc: MockMvc
    @MockitoBean private lateinit var myService: MyService

    @Test
    fun `action should return expected result when condition`() { ... }
}
```
- Test names: backtick strings — `action should result when condition`
- Mockito-Kotlin DSL: `whenever(...).thenReturn(...)`, `verify(service).method()`
- For pure Kotlin logic without Spring context, prefer MockK (Kotlin-native)
- Async assertions: `verify(service, timeout(1000)).method()`
- Coroutine tests: `runTest { ... }`

### Integration Tests (adapters, repositories, external APIs)
- Use Testcontainers for real PostgreSQL and Redis — never mock the database
- Use an abstract base class that starts WireMock and resets it in `@AfterEach`
- Verify outbound requests: `wireMockServer.verify(putRequestedFor(urlEqualTo(...)))`
- Use `@Transactional` rollback on DB integration tests to keep state clean
- Consider property-based testing for financial calculations where edge cases are numerous

### Test Fixtures
- Add builders/factories to a shared fixture file rather than building complex objects inline in each test

### Coverage Requirement
Every public method: happy path + at least one error/edge case.

---

## Observability

### Metrics
```kotlin
Counter.builder("{org}.{domain}.{action}")
    .description("...")
    .tags(Tags.of("result", result))
    .register(meterRegistry)
```
- Naming: `{org}.{domain}.{action}`
- Tag with at least `result` (success/failure)
- Register a metrics event listener on every HTTP client (e.g. `OkHttpMetricsEventListener`)

### Logging
```kotlin
private val logger = KotlinLogging.logger {}

logger.info { "Message with $variable" }
logger.warn(ex) { "Failed to do X for id=$id" }
```
- Always use lambda form `{ }` — avoids string construction when log level is disabled
- Include correlation ID and transaction/entity IDs in every log message
- Never log PII or payment card data

---

## Checklist Before Submitting

- [ ] Test written and watched fail before implementation
- [ ] No `@Autowired` — primary constructor injection only
- [ ] External calls wrapped in resilience executor
- [ ] HTTP errors mapped to domain exceptions at adapter boundary
- [ ] `@Transactional` scope does not span external API calls
- [ ] Currency values use `BigDecimal`; financial operations are idempotent
- [ ] No PII or card data in logs or error messages
- [ ] Flyway migration is backward-compatible and zero-downtime
- [ ] New unbounded queries have a covering index
- [ ] Metrics added for new external calls
- [ ] Logging uses `KotlinLogging` lambda form
- [ ] New config uses `@ConfigurationProperties` data class
- [ ] OpenAPI docs updated for new/changed endpoints

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
