---
name: springboot-code-review
description: "Review Java Spring Boot applications for correctness, security, maintainability, performance, and test coverage. Use when reviewing Spring Boot pull requests, services, controllers, repositories, configuration, or API changes."
---

# Spring Boot Code Review

Perform a risk-focused review of Java Spring Boot code. Prioritize defects, security issues, data loss, incorrect API behavior, transaction problems, and operational failures over style preferences.

## Review workflow

1. Identify the changed behavior and its callers before judging the implementation.
2. Inspect nearby tests, configuration, database migrations, and API contracts when they affect the change.
3. Trace input data through the controller, service, persistence, messaging, and external-client boundaries.
4. Report only findings that are actionable and supported by code evidence.
5. Check whether the change has a focused test that would fail if the defect returned.
6. Run the narrowest available validation command after the review when the project provides one.

## Review priorities

### Correctness and API behavior

- Verify HTTP methods, status codes, request validation, response shapes, and null handling.
- Check pagination, filtering, sorting, and default values for boundary cases.
- Confirm that DTOs are used at API boundaries instead of exposing JPA entities directly.
- Check that state-changing operations are idempotent where retries are possible.
- Look for missing not-found, conflict, authorization, and dependency-failure handling.
- Check backward compatibility for public endpoints, events, and serialized payloads.

### Spring architecture

- Keep controllers focused on transport concerns and services focused on business rules.
- Prefer constructor injection with required dependencies declared as `private final`.
- Check that feature boundaries remain coherent and that business logic is not duplicated.
- Look for incorrect bean scopes, circular dependencies, accidental component scanning, and configuration that is environment-specific but hardcoded.
- Prefer type-safe `@ConfigurationProperties` for grouped configuration.

### Transactions and persistence

- Verify transaction boundaries around each business operation that must be atomic.
- Check propagation, read-only usage, rollback behavior, and self-invocation limitations of `@Transactional`.
- Look for N+1 queries, unbounded result sets, inefficient entity graphs, and incorrect lazy loading.
- Check entity relationships, cascade settings, orphan removal, optimistic locking, and unique constraints.
- Confirm that schema migrations are backward-compatible and that indexes support the new access patterns.
- Check for updates or deletes that can affect unintended rows.

### Security

- Check authentication and authorization at every sensitive endpoint and service operation.
- Verify object-level authorization so users cannot access another user's resource by changing an identifier.
- Ensure passwords and tokens are never logged, hardcoded, or returned in responses.
- Check validation and parameterization against injection, unsafe deserialization, mass assignment, and path traversal.
- Review CORS, CSRF, security headers, actuator exposure, error detail, and secret handling.
- Check rate limiting, replay protection, and secure handling of uploaded files where relevant.

### Reliability and integrations

- Verify timeouts for database, HTTP, messaging, and other external dependencies.
- Check retry behavior for idempotency, backoff, retry limits, and transient versus permanent failures.
- Look for missing circuit breakers, bulkheads, dead-letter handling, or recovery paths where appropriate.
- Check that asynchronous consumers are idempotent and acknowledge messages only after successful processing.
- Confirm resource cleanup for streams, files, transactions, and other closeable resources.

### Observability and operations

- Prefer structured, parameterized logging with useful identifiers and without sensitive data.
- Check correlation IDs, metrics, tracing, health checks, and meaningful failure signals for new flows.
- Verify that exceptions preserve useful context without exposing internal details to API clients.
- Check configuration defaults, profile behavior, startup validation, and production-safe actuator exposure.

### Tests

- Check happy paths, validation failures, authorization failures, not-found cases, conflicts, retries, and transaction boundaries.
- Prefer behavior-focused tests over implementation-detail assertions.
- Use `@WebMvcTest` for controller behavior, `@DataJpaTest` for repository behavior, and `@SpringBootTest` or Testcontainers for integration boundaries when appropriate.
- Check that tests use realistic serialization, database constraints, and security configuration for the behavior under review.
- Treat missing tests as a finding when the change has meaningful regression risk.

## Finding format

Report findings in descending severity using this format:

```text
[P1] Short title
File: path/to/File.java:42
Problem: Explain the concrete failure and the conditions that trigger it.
Impact: Describe the user, data, security, or operational consequence.
Fix: Give the smallest safe correction and identify a regression test when useful.
```

Use these priorities:

- `P0`: Critical security issue, data loss, corruption, or outage requiring immediate action.
- `P1`: High-impact defect likely to affect normal production usage or an important security boundary.
- `P2`: Defect with limited scope, a significant reliability or maintainability risk, or missing important coverage.
- `P3`: Minor issue that is still actionable and worth addressing.

Do not report formatting preferences, speculative risks without a plausible trigger, or issues unrelated to the reviewed change. If no actionable findings are identified, say so clearly and list the remaining test or environment gaps.

## Validation commands

Use the build tool and project conventions. Typical commands are:

```bash
./mvnw test
./mvnw verify
./gradlew test
./gradlew check
```

For a focused change, prefer the narrowest relevant test or static-analysis task first. Do not claim a review is verified unless the command output was actually checked.