---
name: python-code-review
description: "Review Python applications (Django, Flask, FastAPI, or plain Python) for correctness, security, maintainability, performance, and test coverage. Use when reviewing Python pull requests, views, services, or API changes."
---

# Python Code Review

Perform a risk-focused review of Python code. Prioritize defects, security issues, data loss, incorrect API behavior, and operational failures over style preferences.

## Review workflow

1. Identify the changed behavior and its callers before judging the implementation.
2. Inspect nearby tests, configuration, migrations, and API contracts when they affect the change.
3. Trace input data through the view/route, service, data-access, and external-client boundaries.
4. Report only findings that are actionable and supported by code evidence.
5. Check whether the change has a focused test that would fail if the defect returned.
6. Run the narrowest available validation command after the review when the project provides one.

## Review priorities

### Correctness and API behavior

- Verify HTTP methods, status codes, request validation (Pydantic, DRF serializers, WTForms), and response shapes.
- Check pagination, filtering, sorting, and default/mutable-default-argument pitfalls.
- Confirm exception handling is specific (avoid bare `except:`), and that errors are not silently swallowed.
- Check that state-changing operations are idempotent where retries are possible.
- Look for missing not-found, conflict, authorization, and dependency-failure handling.
- Check backward compatibility for public endpoints and serialized payloads.

### Architecture and code quality

- Keep views/routes focused on transport concerns and services/managers focused on business rules.
- Check for correct use of type hints and consistency with `mypy`/`pyright` if configured.
- Look for blocking synchronous calls in async code paths (`async def` functions calling blocking I/O) that should use `await`/async clients.
- Check for proper context manager usage (`with`) for files, locks, and connections.

### Data access and persistence

- Verify transaction boundaries around operations that must be atomic (Django `atomic()`, SQLAlchemy sessions).
- Look for N+1 queries (Django ORM `select_related`/`prefetch_related`, SQLAlchemy lazy loading), unbounded result sets, and missing indexes for new query patterns.
- Check that raw SQL and ORM usage is parameterized and not vulnerable to injection (no string-formatted queries).
- Confirm migrations are backward-compatible and reversible where the framework expects it.

### Security

- Check authentication and authorization (decorators, permission classes, middleware) at every sensitive endpoint.
- Verify object-level authorization so users cannot access another user's resource by changing an identifier.
- Ensure passwords, tokens, and secrets are never logged, hardcoded, or returned in responses; check `.env`/settings usage vs. committed secrets.
- Check validation and sanitization against injection (SQL, command), unsafe deserialization (`pickle`, `yaml.load` without `SafeLoader`, `eval`/`exec` on untrusted input), and path traversal.
- Review CORS configuration, CSRF protection (especially Django), security headers, `DEBUG` mode, and error responses that may leak stack traces or `SECRET_KEY`.
- Check dependency risk for known-vulnerable packages and unpinned versions.
- Check rate limiting and secure handling of uploaded files where relevant.

### Reliability and integrations

- Verify timeouts for database, HTTP clients (`requests`, `httpx`), and other external dependencies.
- Check retry behavior for idempotency, backoff, and retry limits.
- Look for missing circuit breakers, dead-letter handling, or recovery paths where appropriate (Celery tasks, queue consumers).
- Check that task/queue consumers are idempotent and acknowledge messages only after successful processing.
- Confirm resource cleanup for files, sockets, and database connections/sessions.

### Observability and operations

- Prefer structured logging with useful identifiers and without sensitive data; avoid bare `print()` for operational logging.
- Check correlation/request IDs, metrics, tracing, health checks, and meaningful failure signals for new flows.
- Verify that exceptions preserve useful context without exposing internal details to API clients.
- Check configuration defaults, environment-specific behavior, and startup validation.

### Tests

- Check happy paths, validation failures, authorization failures, not-found cases, conflicts, retries, and error branches.
- Prefer behavior-focused tests over implementation-detail assertions (`pytest`, `unittest`).
- Use fixtures/factories realistically and check that integration tests exercise real serialization, database constraints, and auth configuration.
- Treat missing tests as a finding when the change has meaningful regression risk.

## Finding format

Report findings in descending severity using this format:

```text
[P1] Short title
File: path/to/file.py:42
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

Use the project's tooling and conventions. Typical commands are:

```bash
pytest
python -m pytest
tox
ruff check .
mypy .
```

For a focused change, prefer the narrowest relevant test or lint task first. Do not claim a review is verified unless the command output was actually checked.
