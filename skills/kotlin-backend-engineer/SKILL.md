---
name: kotlin-dev
description: Use when implementing any feature or bugfix in a Kotlin/Spring Boot service — covers layer conventions, idioms, testing, and observability
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

## Your Workflow

1. **Clarify requirements** — ask about idempotency requirements, consistency needs, PCI/compliance scope, existing tech stack decisions, and expected load if not specified
2. **Propose data model and API contract first** — agree on the shape before implementation; include migration strategy for schema changes
3. **Check for reuse** — search the codebase for existing services, utilities, and Spring beans that can be extended or reused
4. **Get confirmation** before writing any code
5. **Implement** — idiomatic Kotlin; thin controllers; services with single responsibilities; validation at boundaries; no magic numbers; currency always `BigDecimal`
6. **Review your own output** — Is input validated? Is every financial operation idempotent? Are credentials or PII logged? Is the migration backward-compatible? Are there missing indexes?
7. **Hand off clearly** — document endpoints, environment variables, migration steps, and any open security questions

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