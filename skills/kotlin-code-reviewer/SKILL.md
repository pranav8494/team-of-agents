---
name: kotlin-reviewer
description: Use when reviewing a Kotlin/Spring Boot pull request — systematic checklist covering architecture, idioms, testing, security, and observability
version: 2.1.0
---

# Kotlin Code Reviewer Skill

## Iron Law

```
Block any PR that violates architectural boundaries, skips tests, or introduces security issues.
Flag (not block) style deviations that don't affect correctness.
```

---

## Before Taking Any Action

1. **Announce** that you are beginning a code review and note the scope (file names, PR description if provided)
2. **Ask for context** if not provided: what is this code doing, are there related PRs, are there known constraints?
3. **Present findings** as a structured review, not a stream of consciousness
4. **Note explicitly** any areas you could not fully assess (e.g. missing context, untestable logic without runtime data)

---

## Your Workflow

1. **Read the PR description or stated intent** — understand what the author was trying to achieve
2. **Run all five phases in order** — complete all phases before commenting
3. **Scan for blockers first** — security issues, data correctness bugs, transaction problems get flagged before anything else
4. **Check test coverage** — are new/changed code paths tested? are edge cases covered?
5. **Review Kotlin/Java idioms** — is the code idiomatic? null-safety hazards? coroutine misuse?
6. **Flag style and nitpicks last** — only if they affect readability meaningfully
7. **Write the summary** — overall verdict + 2-3 sentences on what the PR does well and where the main concerns are

---

## Review Process

**Phase 1 — Architecture boundaries** (block if violated)  
**Phase 2 — Kotlin correctness** (block if violated)  
**Phase 3 — Testing adequacy** (block if violated)  
**Phase 4 — Observability** (flag if missing)  
**Phase 5 — Security** (block if violated)

---

## Phase 1 — Architecture Boundaries

### Controller
- [ ] Zero business logic — only calls service, returns response
- [ ] Primary constructor injection — no `@Autowired`, no field injection
- [ ] `@ResponseStatus(HttpStatus.NO_CONTENT)` on delete endpoints
- [ ] Input validated with Bean Validation before reaching service layer

### Service
- [ ] All external calls go through resilience executor — no direct client calls
- [ ] `@Cacheable` / `@CacheEvict` used for repeated reads of stable data
- [ ] Concurrent I/O uses `async`/`awaitAll()`, not sequential calls
- [ ] Fire-and-forget coroutines catch all exceptions and log — no silent failures
- [ ] `@Transactional` scope does not span external API calls

### Adapter / Client
- [ ] HTTP errors mapped to domain exceptions at adapter boundary — not in service or controller
- [ ] New HTTP clients have metrics instrumentation

### Repository
- [ ] N+1 queries absent — fetch strategies reviewed; no unbounded queries without a covering index
- [ ] JPA mappings correct: lazy vs eager, cascade behaviour, orphan removal

### Mapper
- [ ] Mappers are `object` with extension functions — not Spring beans
- [ ] Missing data returns `null`, not throws — call sites use `mapNotNull`

### Exception
- [ ] New exceptions extend the base HTTP exception class with correct `HttpStatus`
- [ ] Sealed result types used for expected failure paths — exceptions reserved for unexpected conditions

### Configuration
- [ ] New config uses `@ConfigurationProperties` data class with constructor defaults
- [ ] No secrets hardcoded in `application.yml` or config classes

---

## Phase 2 — Kotlin Correctness

### Reject these patterns

| Anti-pattern | Required replacement |
|---|---|
| `if (x != null) { use(x) }` | `x?.let { use(it) }` |
| `if (x != null) x else default` | `x ?: default` |
| Manual `for` loop over collection | `map`, `filter`, `mapNotNull`, `forEach` |
| Building a map with `.put()` in a loop | `associateBy { }` or `groupBy { }` |
| `@Autowired lateinit var` in production code | Primary constructor `private val` |
| Mutable `var` where `val` works | `val` |
| `logger.info("msg $var")` (eager string) | `logger.info { "msg $var" }` (lazy lambda) |
| `logger.error("$exception")` | `logger.error(exception) { "context" }` |
| Nullable collection return for "empty" | Return `emptyList()` / `emptyMap()` |
| `!!` operator without a documented invariant | Safe call or explicit check |

### Data classes
- [ ] DTOs and domain models are `data class`
- [ ] No `var` fields in `data class` unless mutation is genuinely required

### Coroutines
- [ ] `suspend` functions do not block — no `Thread.sleep()`, no blocking I/O without `Dispatchers.IO`
- [ ] Structured concurrency respected — no fire-and-forget without explicit scope and error handling
- [ ] Cancellation handled — no `CancellationException` swallowed

### Fintech domain
- [ ] Currency values use `BigDecimal` — never `Double` or `Float`
- [ ] Financial operations are idempotent — idempotency key present and enforced
- [ ] Audit log entries are immutable — no updates, only appends
- [ ] Transaction isolation level appropriate for the financial operation

---

## Phase 3 — Testing Adequacy

### Coverage (block if missing)
- [ ] Every new public method has a test
- [ ] Happy path covered
- [ ] At least one error/exception path covered
- [ ] Edge cases covered (empty input, null, error responses from dependencies)

### Test quality
- [ ] Test names use backtick strings: `` `action should result when condition` ``
- [ ] Mockito-Kotlin DSL used: `whenever`, `verify` — no raw `Mockito.when()`
- [ ] `@MockitoBean` for Spring context mocks; `mock<T>()` for pure unit mocks
- [ ] Async assertions use `verify(service, timeout(N))` — no `Thread.sleep()`
- [ ] Coroutine tests use `runTest { }` from `kotlinx.coroutines.test`
- [ ] Complex domain objects built via fixture factories, not inline

### Integration tests
- [ ] Adapter tests use the project's abstract WireMock base class
- [ ] `@AfterEach` resets WireMock state (inherited — not duplicated)
- [ ] DB integration tests use Testcontainers (real PostgreSQL) — never mock the database
- [ ] Flyway migrations reviewed: backward-compatible, forward-only, zero-downtime

---

## Phase 4 — Observability

### Metrics (flag if missing)
- [ ] New external calls have a counter or timer
- [ ] Metric name follows the project convention (`{org}.{domain}.{action}`)
- [ ] Tagged with at least `result` (success/failure)
- [ ] HTTP clients registered with metrics event listener

### Logging (flag if wrong)
- [ ] One `private val logger = KotlinLogging.logger {}` per class
- [ ] All log calls use lambda form
- [ ] Exceptions passed as first arg: `logger.error(e) { "context" }`
- [ ] Log messages include relevant IDs (correlation ID, entity ID); no PII

---

## Phase 5 — Security

### Block on any of
- [ ] Hardcoded secrets, tokens, or credentials
- [ ] Auth/permission checks bypassed or silently ignored
- [ ] User input interpolated directly into search/DB queries (injection risk)
- [ ] PII or tokens in log messages
- [ ] Financial/cryptographic logic changed — flag for extra scrutiny even if it looks correct

---

## Commenting Guidelines

**Severity labels — every comment must have one:**
- `[blocker]` — must fix before merge (architecture violation, missing test, security issue)
- `[major]` — should fix (significant idiom problem, missing observability)
- `[minor]` — fix if easy (style that affects readability)
- `[nit]` — optional style preference
- `[question]` — seeking clarification before judging
- `[nice]` — positive feedback on a well-written test, clean abstraction, or elegant Kotlin

**Format rules:**
- Every comment explains: the problem, the risk, and a concrete suggestion or code example
- Group comments by phase, not by file order
- One clear comment per issue — don't scatter the same concern across multiple inline comments
- Praise good work — clean abstractions and well-written tests earn a `[nice]`

**Overall verdict:**
- **Approve** — no issues
- **Approve with minor comments** — trivial items that don't block merge
- **Request Changes** — major or minor issues that need addressing
- **Block** — blocker-level security or correctness issue