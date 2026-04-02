# Backend Best Practices — Java 17 + Spring Boot

<!-- VARIANT FILE: Seeded from the OBNS/ESAP project conventions.
     Copy this file to context/best-practices/backend.md in your project.
     Then edit it to match your project's specific conventions, module names,
     and invariants. Remove OBNS-specific sections that don't apply.
-->

---

## Mandatory Conventions

1. **Constructor injection only** — never `@Autowired` on fields; use `@RequiredArgsConstructor` or an explicit constructor
2. **`TimeZone.setDefault(TimeZone.getTimeZone("UTC"))`** in every `main()` method
3. **`@ConfigurationProperties(prefix = "...")`** on config classes — no `@Value` and no magic strings in beans
4. **Failsafe retry:** `RetryPolicy` with `withMaxRetries()` + `withBackoff()` — **never `withMaxDuration()`**
5. **DB schema changes via migration tool only** (Liquibase) — never `spring.jpa.hibernate.ddl-auto` beyond `validate`
6. **REST paths:** `/v1/{domain}api/{resources}` (external) / `/v1/{domain}-internal/{resources}` (internal)
7. **OpenAPI annotations** on all external controllers: `@Operation`, `@Tag`, `@ApiResponse`, `@SecurityRequirement`
8. **Conditional beans** via `@ConditionalOnProperty` — never hardcode environment forks in code
9. **No version numbers** on individual `<dependency>` entries — all versions managed in the root POM / BOM
10. **`@Import({...})`** for shared Spring configs in every application class

---

## Dependency Injection Pattern

```java
// CORRECT
@Service
@RequiredArgsConstructor
public class MyService {
    private final MyRepository repository;
    private final MyConfiguration config;
}

// WRONG — never do this
@Service
public class MyService {
    @Autowired
    private MyRepository repository;
}
```

---

## Configuration Pattern

```java
// MyConfiguration.java
@Configuration
@ConfigurationProperties(prefix = "myapp")
@Validated
@Data
public class MyConfiguration {
    @Valid
    private Retry retry = new Retry();

    @Data
    public static class Retry {
        private int maxAttempts = 3;
        private long initialIntervalSeconds = 2;
        private long maxIntervalMinutes = 10;
    }
}

// application.yml
myapp:
  retry:
    max-attempts: 3
    initial-interval-seconds: 2
    max-interval-minutes: 10
```

---

## Failsafe Retry Pattern

```java
RetryPolicy<T> policy = RetryPolicy.<T>builder()
    .handle(SpecificException.class)
    .withMaxRetries(config.getRetry().getMaxAttempts())
    .withBackoff(
        Duration.ofSeconds(config.getRetry().getInitialIntervalSeconds()),
        Duration.ofMinutes(config.getRetry().getMaxIntervalMinutes())
    )
    // NEVER use .withMaxDuration() — it causes silent failures under load
    .build();

Failsafe.with(policy).run(() -> operation());
```

---

## Liquibase Migration Pattern

```xml
<!-- db/changelog/V2025.01.15__add-my-column.xml -->
<changeSet id="MYAPP-123-add-my-column" author="yourname">
    <addColumn tableName="my_table">
        <column name="my_column" type="VARCHAR(100)">
            <constraints nullable="true"/>
        </column>
    </addColumn>
</changeSet>
```

- Naming: `V{YYYY.MM.DD}__{description}.xml`
- changeSet id must be unique and reference the ticket
- Migrations must be idempotent

---

## REST Controller Pattern

```java
@RestController
@RequestMapping("/v1/mydomainapi")
@RequiredArgsConstructor
@Tag(name = "My Domain API")
public class MyController {

    private final MyService service;

    @GetMapping("/{id}")
    @Operation(summary = "Get item by ID")
    @ApiResponse(responseCode = "200", description = "Item found")
    @ApiResponse(responseCode = "404", description = "Item not found")
    @SecurityRequirement(name = "bearer-key")
    public ResponseEntity<MyDto> getById(@PathVariable Long id) {
        return ResponseEntity.ok(service.getById(id));
    }
}
```

---

## Error Handling Pattern

```java
// DO: catch specific exceptions and map to meaningful errors
try {
    service.process(submission);
} catch (ValidationException e) {
    errorHandler.handleValidationError(submission.getId(), e);
} catch (CommunicationException e) {
    errorHandler.handleCommunicationError(submission.getId(), e);
}

// DON'T: swallow exceptions
try {
    service.process(submission);
} catch (Exception e) {
    log.error("error", e); // then continue — this hides failures
}
```

---

## Testing Conventions

### Test Types

| Type | Annotation | DB | Spring Context |
|------|-----------|-----|---------------|
| Unit | `@ExtendWith(MockitoExtension.class)` | None | No |
| Integration | `@SpringBootTest @ActiveProfiles("test")` | H2 in-memory | Yes |

### Coverage Targets

- **Line coverage:** ≥ 80%
- **Branch coverage:** ≥ 70%

### Unit Test Structure

```java
@ExtendWith(MockitoExtension.class)
class MyServiceTest {

    @Mock
    private MyRepository repository;

    @Mock
    private MyConfiguration config;

    @InjectMocks
    private MyService underTest;

    @BeforeEach
    void setUp() {
        // per-test setup
    }

    @Test
    void methodName_withCondition_expectedBehaviour() {
        // Arrange
        when(repository.findById(1L)).thenReturn(Optional.of(new MyEntity()));

        // Act
        MyResult result = underTest.method(1L);

        // Assert
        assertThat(result).isNotNull();
        verify(repository).findById(1L);
    }

    @Test
    void methodName_withNullInput_throwsIllegalArgumentException() {
        // Act & Assert
        assertThatThrownBy(() -> underTest.method(null))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessageContaining("id cannot be null");
    }
}
```

### Integration Test Structure

```java
@SpringBootTest
@ActiveProfiles("test")
class MyServiceIntegrationTest {

    @Autowired
    private MyService underTest;

    @Autowired
    private MyRepository repository;

    @Test
    void method_persistsCorrectly() {
        // Arrange
        MyEntity entity = repository.save(new MyEntity());

        // Act
        underTest.method(entity.getId());

        // Assert
        MyEntity updated = repository.findById(entity.getId()).orElseThrow();
        assertThat(updated.getStatus()).isEqualTo(ExpectedStatus.DONE);
    }
}
```

### Test Naming Convention

`methodName_withCondition_expectedBehaviour`

Examples:
- `process_withValidInput_createsSubmission`
- `process_withMissingField_throwsValidationException`
- `updateState_withNullId_throwsIllegalArgumentException`

### Assertion Library

Use AssertJ (`assertThat(...)`) — not JUnit `assertEquals`. For exception assertions use `assertThatThrownBy`.

---

## Logging Convention

```java
// Business events → INFO
log.info("[process] Starting submission id={}", id);

// Diagnostic / debugging → DEBUG (not committed at INFO level)
log.debug("[process] Metadata resolved: {}", metadata);

// Failures → ERROR with exception
log.error("[process] Communication failed for id={}", id, e);

// NEVER log sensitive data (credentials, PII, security tokens)
```

---

## Static Analysis

- Suppress Coverity false positives with `/* coverity[...] */` comments when justified
- Address all SonarQube issues before merge unless annotated with a tracked suppression
