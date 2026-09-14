---
name: nodejs-code-review
description: "Review Node.js applications (Express, Fastify, NestJS, or plain Node) for correctness, security, maintainability, performance, and test coverage. Use when reviewing Node.js/TypeScript pull requests, routes, controllers, services, or API changes."
---

# Node.js Code Review

Perform a risk-focused review of Node.js/TypeScript code. Prioritize defects, security issues, data loss, incorrect API behavior, and operational failures over style preferences.

## Review workflow

1. Identify the changed behavior and its callers before judging the implementation.
2. Inspect nearby tests, environment configuration, and API contracts when they affect the change.
3. Trace input data through the route/controller, service, data-access, and external-client boundaries.
4. Report only findings that are actionable and supported by code evidence.
5. Check whether the change has a focused test that would fail if the defect returned.
6. Run the narrowest available validation command after the review when the project provides one.

## Review priorities

### Correctness and API behavior

- Verify HTTP methods, status codes, request validation (e.g. `zod`, `joi`, `class-validator`), and response shapes.
- Check pagination, filtering, sorting, and default values for boundary cases.
- Confirm async/await and Promise chains handle rejections; look for unhandled promise rejections and missing `try/catch`.
- Check that state-changing operations are idempotent where retries are possible.
- Look for missing not-found, conflict, authorization, and dependency-failure handling.
- Check backward compatibility for public endpoints and serialized payloads.

### Architecture and code quality

- Keep route/controller layers focused on transport concerns and services focused on business rules.
- Check for correct dependency injection/module boundaries (NestJS modules, Express routers, etc.).
- Look for callback-style code that should be promisified, and mixed use of callbacks and async/await.
- Check for proper use of `const`/`let`, avoidance of unnecessary `any` in TypeScript, and strict typing at boundaries.
- Look for blocking synchronous calls (e.g. `fs.readFileSync`, heavy CPU loops) on the event loop in request-handling paths.

### Data access and persistence

- Verify transaction boundaries around operations that must be atomic (e.g. Prisma `$transaction`, Sequelize transactions, Mongoose sessions).
- Look for N+1 queries, unbounded result sets, and missing indexes for new query patterns.
- Check that ORM/query-builder usage is parameterized and not vulnerable to injection.
- Confirm migrations are backward-compatible.

### Security

- Check authentication and authorization (middleware/guards) at every sensitive route.
- Verify object-level authorization so users cannot access another user's resource by changing an identifier.
- Ensure passwords, tokens, and secrets are never logged, hardcoded, or returned in responses; check `.env` usage vs. committed secrets.
- Check validation and sanitization against injection (SQL/NoSQL), XSS, prototype pollution, and unsafe deserialization (`eval`, `JSON.parse` on untrusted input, `child_process` with unsanitized input).
- Review CORS configuration, security headers (helmet), CSRF protection, and error responses that may leak stack traces.
- Check dependency risk for known-vulnerable packages and unpinned versions.
- Check rate limiting and secure handling of uploaded files where relevant.

### Reliability and integrations

- Verify timeouts for database, HTTP clients (axios/fetch), and other external dependencies.
- Check retry behavior for idempotency, backoff, and retry limits.
- Look for missing circuit breakers, dead-letter handling, or recovery paths where appropriate.
- Check that queue/event consumers are idempotent and acknowledge messages only after successful processing.
- Confirm resource cleanup for streams, file handles, and open connections.
- Check for proper process-level error handling (`unhandledRejection`, `uncaughtException`) without silently swallowing errors.

### Observability and operations

- Prefer structured, parameterized logging (e.g. `pino`, `winston`) with useful identifiers and without sensitive data.
- Check correlation/request IDs, metrics, tracing, health checks, and meaningful failure signals for new flows.
- Verify that errors preserve useful context without exposing internal details to API clients.
- Check configuration defaults, environment-specific behavior, and startup validation.

### Tests

- Check happy paths, validation failures, authorization failures, not-found cases, conflicts, retries, and error branches.
- Prefer behavior-focused tests over implementation-detail assertions (Jest/Vitest/Mocha).
- Check that integration tests use realistic serialization, database constraints, and auth configuration for the behavior under review.
- Treat missing tests as a finding when the change has meaningful regression risk.

## Finding format

Report findings in descending severity using this format:

```text
[P1] Short title
File: path/to/file.ts:42
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

Use the project's package manager and scripts. Typical commands are:

```bash
npm test
npm run lint
npm run build
yarn test
pnpm test
```

For a focused change, prefer the narrowest relevant test or lint task first. Do not claim a review is verified unless the command output was actually checked.
