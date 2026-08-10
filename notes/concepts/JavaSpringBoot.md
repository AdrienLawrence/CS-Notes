# Overview

- Spring is a Java ecosystem centered on dependency injection, application configuration, web development, data access, security, testing, and integrations

- Spring Boot is an opinionated layer on Spring which provides automatic configuration, starter dependencies, embedded servers, production tooling, and executable applications

- Spring Framework supplies the core features. Spring Boot makes them easier to configure and run

- A typical web request passes through:

    Client -> Filters/Security -> Controller -> Service -> Repository -> Database

- Common responsibilities:

    Controller: Translate HTTP input and output

    Service: Business rules and transaction boundaries

    Repository: Data access

    Entity: Persistence model

    DTO: Public request or response model

- Spring Boot versions have specific Java and build-tool requirements. Check the official system requirements and the project's parent/plugin version before starting

# Creating and Running a Project

- Spring Initializr (`start.spring.io`) generates Maven or Gradle projects with selected dependencies

- Common starters:

    `spring-boot-starter-web`: Spring MVC, JSON, validation integration, embedded servlet server

    `spring-boot-starter-webflux`: Reactive web stack

    `spring-boot-starter-data-jpa`: JPA and Hibernate integration

    `spring-boot-starter-security`: Authentication and authorization

    `spring-boot-starter-validation`: Jakarta Bean Validation

    `spring-boot-starter-test`: JUnit, Spring Test, AssertJ, Mockito, and related testing tools

    `spring-boot-starter-actuator`: Production health and metrics endpoints

- Main application:

    ```java
    package com.example.demo;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;

    @SpringBootApplication
    public class DemoApplication {
        public static void main(String[] args) {
            SpringApplication.run(DemoApplication.class, args);
        }
    }
    ```

- `@SpringBootApplication` combines configuration, component scanning, and auto-configuration

- Place the main class in a root package above application components so component scanning finds them

- Common commands:

    ```bash
    ./mvnw spring-boot:run
    ./mvnw test
    ./mvnw package
    java -jar target/app.jar
    ```

    ```bash
    ./gradlew bootRun
    ./gradlew test
    ./gradlew bootJar
    ```

# Dependency Injection and Beans

- The Spring container creates and connects managed objects called beans

- Dependency injection means a class declares what it requires instead of constructing dependencies itself

    ```java
    @Service
    public class UserService {
        private final UserRepository repository;

        public UserService(UserRepository repository) {
            this.repository = repository;
        }
    }
    ```

- Constructor injection is preferred because dependencies are explicit, required fields may be final, and the class is easy to test

- Common stereotype annotations:

    `@Component`: General managed component

    `@Service`: Business service

    `@Repository`: Data access component and persistence exception translation

    `@Controller`: MVC controller returning views

    `@RestController`: Controller whose methods normally write response bodies

    `@Configuration`: Class declaring configuration and beans

- A `@Bean` method registers an object which cannot or should not be discovered through component scanning

    ```java
    @Configuration
    public class ClockConfig {
        @Bean
        Clock clock() {
            return Clock.systemUTC();
        }
    }
    ```

- Beans are singleton-scoped by default. Singleton does not automatically mean thread-safe; avoid request-specific mutable fields

- Use `@Qualifier` or `@Primary` when multiple beans satisfy one interface

- Circular dependencies usually indicate responsibilities should be separated

# Configuration and Profiles

- Configuration may come from `application.properties`, `application.yml`, environment variables, command-line arguments, and external sources

    ```yaml
    server:
      port: 8080

    app:
      page-size: 50
      public-url: https://example.com
    ```

- Bind related settings to a validated type with `@ConfigurationProperties`

    ```java
    @ConfigurationProperties(prefix = "app")
    @Validated
    public record AppProperties(
        @Min(1) int pageSize,
        @NotNull URI publicUrl
    ) {}
    ```

- Enable properties through configuration scanning or explicit registration

- Profiles select environment-specific beans and configuration, such as `dev`, `test`, and `prod`

- `application-prod.yml` adds production profile values. Activate with `SPRING_PROFILES_ACTIVE=prod`

- Do not package production secrets in the application. Supply them through protected environment or secret-management systems

- Spring property precedence is defined. Environment and command-line values commonly override packaged defaults

# Spring MVC and REST Controllers

- Spring MVC is the traditional servlet-based web framework

    ```java
    @RestController
    @RequestMapping("/api/users")
    public class UserController {
        private final UserService service;

        public UserController(UserService service) {
            this.service = service;
        }

        @GetMapping("/{id}")
        public UserResponse findById(@PathVariable long id) {
            return service.findById(id);
        }

        @PostMapping
        public ResponseEntity<UserResponse> create(
                @Valid @RequestBody CreateUserRequest request) {
            UserResponse created = service.create(request);
            URI location = URI.create("/api/users/" + created.id());
            return ResponseEntity.created(location).body(created);
        }
    }
    ```

- Mapping annotations include `@GetMapping`, `@PostMapping`, `@PutMapping`, `@PatchMapping`, and `@DeleteMapping`

- Input annotations:

    `@PathVariable`: URL path segment

    `@RequestParam`: Query or form parameter

    `@RequestHeader`: Header

    `@CookieValue`: Cookie

    `@RequestBody`: Deserialized request body

    `MultipartFile`: Uploaded file

- `ResponseEntity` controls body, status, and headers

- Jackson serializes Java values to JSON and deserializes request JSON. Configure date/time and naming behavior deliberately

- Controllers should remain thin. They should not contain database queries or large business workflows

# Validation and Error Handling

- Jakarta Bean Validation annotations define input constraints

    ```java
    public record CreateUserRequest(
        @NotBlank @Size(max = 100) String name,
        @NotBlank @Email String email,
        @Min(18) int age
    ) {}
    ```

- Add `@Valid` to validate a body or nested object. Use `@Validated` for method constraints and validation groups

- Validation must occur at the system boundary even when the frontend already validates input

- Centralize HTTP error translation with `@RestControllerAdvice`

    ```java
    @RestControllerAdvice
    public class ApiExceptionHandler {
        @ExceptionHandler(UserNotFoundException.class)
        ResponseEntity<ProblemDetail> handleNotFound(UserNotFoundException exception) {
            ProblemDetail problem = ProblemDetail.forStatusAndDetail(
                HttpStatus.NOT_FOUND,
                exception.getMessage()
            );
            return ResponseEntity.status(HttpStatus.NOT_FOUND).body(problem);
        }
    }
    ```

- `ProblemDetail` models RFC-compatible HTTP problem responses

- Separate expected domain failures from unexpected bugs. Do not expose stack traces or SQL details to clients

# Service Layer and Transactions

- Services coordinate business behavior across repositories and external systems

- `@Transactional` defines a database transaction boundary

    ```java
    @Service
    public class TransferService {
        @Transactional
        public void transfer(long fromId, long toId, BigDecimal amount) {
            Account from = accounts.getRequired(fromId);
            Account to = accounts.getRequired(toId);
            from.withdraw(amount);
            to.deposit(amount);
        }
    }
    ```

- A runtime exception normally rolls back the transaction. Checked-exception rules can be configured

- Put transaction boundaries on public service methods representing a complete unit of work

- Spring applies many features through proxies. Self-invocation inside the same object may bypass proxy behavior such as `@Transactional`

- Keep transactions short and avoid slow network calls while holding database locks

- Use isolation and locking based on actual consistency requirements, not as automatic defaults

# Spring Data JPA

- JPA is a persistence specification. Hibernate is the common implementation used by Spring Boot

- An entity maps an object identity and fields to database state

    ```java
    @Entity
    @Table(name = "users")
    public class User {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;

        @Column(nullable = false)
        private String name;

        @Column(nullable = false, unique = true)
        private String email;

        protected User() {}
    }
    ```

- A repository interface gains generated CRUD behavior

    ```java
    public interface UserRepository extends JpaRepository<User, Long> {
        Optional<User> findByEmailIgnoreCase(String email);

        @Query("select u from User u where u.name like concat('%', :text, '%')")
        Page<User> search(@Param("text") String text, Pageable pageable);
    }
    ```

- `JpaRepository` supports CRUD, sorting, pagination, flushing, and batch-related operations

- Derived query names are convenient for simple conditions. Use JPQL, specifications, Query by Example, or explicit query code when names become unreadable

- Entity states include transient, managed, detached, and removed

- Dirty checking writes changes made to managed entities when the transaction flushes

- Relationships include `@OneToOne`, `@OneToMany`, `@ManyToOne`, and `@ManyToMany`

- Choose ownership, cascade, orphan removal, and fetch behavior intentionally

- Lazy relationships may cause N+1 queries. Use fetch joins, entity graphs, projections, or purpose-built queries

- Avoid serializing entities directly. Bidirectional relationships, lazy loading, and private columns make them poor API contracts

- Use Flyway or Liquibase migrations for production schema history. `ddl-auto=create` is not a production migration strategy

# DTOs and Mapping

- Request DTOs define accepted client input. Response DTOs define public output

- DTOs protect the API contract from entity structure and prevent mass assignment of server-controlled fields

    ```java
    public record UserResponse(long id, String name) {
        static UserResponse from(User user) {
            return new UserResponse(user.getId(), user.getName());
        }
    }
    ```

- Mapping may be manual or use a compile-time mapper such as MapStruct

- Do not return password hashes, security roles, internal timestamps, or entire related object graphs by accident

# Spring Security

- Spring Security uses a filter chain before controller execution

- Authentication establishes identity. Authorization decides whether the identity may access an action

    ```java
    @Configuration
    @EnableMethodSecurity
    public class SecurityConfig {
        @Bean
        SecurityFilterChain security(HttpSecurity http) throws Exception {
            return http
                .authorizeHttpRequests(auth -> auth
                    .requestMatchers("/actuator/health", "/api/public/**").permitAll()
                    .requestMatchers(HttpMethod.DELETE, "/api/users/**").hasRole("ADMIN")
                    .anyRequest().authenticated()
                )
                .oauth2ResourceServer(oauth -> oauth.jwt(Customizer.withDefaults()))
                .build();
        }
    }
    ```

- The `SecurityFilterChain` configures route rules and authentication mechanisms

- Method rules such as `@PreAuthorize` protect business operations close to their execution

- Passwords must use a `PasswordEncoder` designed for password hashing, not reversible encryption or a fast general hash

- Browser cookie authentication needs CSRF protection. Do not disable CSRF without understanding the client and credential transport

- Configure CORS separately from authentication

- JWT resource servers must verify signatures and important claims. A decoded token is not necessarily a valid token

- Use least privilege and repeat ownership/permission checks inside protected service operations

# Filters, Interceptors, and AOP

- Servlet filters work at the HTTP container level and are appropriate for request wrapping, correlation IDs, and security integration

- Handler interceptors run around controller handling and can inspect the selected handler

- Spring AOP applies cross-cutting behavior around bean method execution

- Common AOP uses include transactions, metrics, auditing, and carefully scoped logging

- Proxy limitations apply: only managed beans and intercepted calls receive advice

- Avoid hiding core business behavior inside broad aspects because execution becomes hard to follow

# Events, Scheduling, and Async Work

- Application events decouple in-process publishers and listeners

- `@TransactionalEventListener` can run based on transaction phase

- `@Scheduled` runs repeated jobs when scheduling is enabled

- `@Async` executes eligible bean methods through an executor when async support is enabled

- In-process events, scheduling, and async methods are not durable queues. Multi-instance applications may execute a scheduled job more than once

- Use a durable broker/job system for retries, ordering, persistence, and independently scaled consumers

# Caching

- Spring's cache abstraction uses annotations such as `@Cacheable`, `@CachePut`, and `@CacheEvict`

    ```java
    @Cacheable(cacheNames = "users", key = "#id")
    public UserResponse findById(long id) {
        return loadUser(id);
    }
    ```

- The abstraction requires a cache provider such as Caffeine or Redis

- Cache keys, expiration, invalidation, serialization, and multi-instance behavior must be designed explicitly

- Never place user-specific or permission-dependent results into a shared cache without the relevant identity and authorization context

# Actuator, Observability, and Logging

- Actuator provides health, info, metrics, environment, log, mapping, and other operational endpoints

- Expose only required endpoints and secure sensitive ones

- Health indicators should distinguish readiness to receive traffic from basic process liveness

- Micrometer provides vendor-neutral metrics. Micrometer Tracing integrates distributed trace context

- Structured logs should include timestamp, severity, service, environment, trace/request ID, and meaningful event fields

- Never log passwords, access tokens, session cookies, or unnecessary personal data

# Testing

- `@SpringBootTest` loads a broad application context and is appropriate for full integration tests

- Slice tests load a focused layer:

    `@WebMvcTest`: MVC controllers

    `@DataJpaTest`: JPA repositories

    `@JsonTest`: JSON serialization

- `MockMvc` tests servlet controller behavior without a live network server

    ```java
    @WebMvcTest(UserController.class)
    class UserControllerTest {
        @Autowired MockMvc mvc;
        @MockBean UserService service;

        @Test
        void returnsUser() throws Exception {
            given(service.findById(1L)).willReturn(new UserResponse(1L, "Alice"));

            mvc.perform(get("/api/users/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value("Alice"));
        }
    }
    ```

- Testcontainers runs real disposable dependencies such as PostgreSQL, Redis, or Kafka for integration tests

- Prefer real integration behavior for persistence and serialization instead of mocking framework internals

- Unit-test business rules as normal Java classes without loading Spring

# WebFlux

- Spring WebFlux is the reactive non-blocking web stack using Reactor types such as `Mono` and `Flux`

- Use WebFlux when the complete request path uses non-blocking libraries and the application benefits from high waiting concurrency or streaming

- Blocking JPA/JDBC calls inside the reactive event loop remove the main benefit and can stall other requests

- Reactive code adds a different programming model and debugging cost. Spring MVC with virtual threads or normal request threads is often simpler for blocking dependencies

- Do not mix MVC and WebFlux accidentally through starter dependencies

# Packaging and Deployment

- Spring Boot creates an executable JAR containing application code and dependencies

- Buildpacks can create OCI container images with `bootBuildImage`

- Production configuration should include memory limits, JVM container awareness, timeouts, connection pools, graceful shutdown, health probes, and observability

- Avoid running database migration independently in every scaled application instance unless the migration tool and deployment process safely coordinate it

- Place a reverse proxy or gateway in front when TLS termination, rate limiting, request limits, or centralized routing are required

- Keep the service stateless where possible. Store durable sessions and files in shared external systems when running multiple instances

# Quick Reference

- Managed object -> Bean

- Required collaborator -> Constructor injection

- HTTP endpoint -> `@RestController`

- Business behavior -> `@Service`

- Database access -> Repository

- Complete database unit of work -> `@Transactional` service method

- Request validation -> DTO plus `@Valid`

- Public response boundary -> Response DTO

- Central HTTP errors -> `@RestControllerAdvice`

- Route security -> `SecurityFilterChain`

- Business permission -> `@PreAuthorize` or explicit service check

- Schema history -> Flyway or Liquibase

- Full integration test -> `@SpringBootTest`

- Focused layer test -> Test slice

- Production health/metrics -> Actuator and Micrometer
