---
name: enterprise-code-quality
description: Enforce enterprise-grade implementation, secure coding, test reliability, and SonarQube quality gates across Java Spring Boot, React/TypeScript, and Python/FastAPI projects.
---

# Enterprise Code Quality Skill

## 1. Purpose and non-negotiable principles

Use this skill whenever implementing, modifying, reviewing, refactoring, or debugging application code.

The goal is **correct, secure, maintainable, testable, and production-ready code**. Do not optimize for merely making a build pass or silencing a static-analysis warning.

### Hard rules

1. Never claim tests, builds, scans, or deployments passed unless they were actually run and their results were inspected.
2. Never claim “zero SonarQube issues” unless the relevant scan completed and the quality gate result was verified.
3. Never hide, suppress, downgrade, or exclude a finding merely to make a dashboard green.
4. Never delete, weaken, skip, or make tests less meaningful to obtain a passing result.
5. Never invent test results, coverage, scan metrics, or CI status.
6. Do not change public APIs, contracts, schemas, behavior, or security controls without understanding impact and documenting the change.
7. If requirements are ambiguous, identify the ambiguity and ask a focused question when it blocks safe implementation. Otherwise state a reasonable assumption.
8. Prefer small, reviewable changes over broad rewrites.
9. Keep secrets, credentials, tokens, personal data, and production data out of source code, logs, tests, prompts, and commits.
10. If a required tool or environment is unavailable, report the check as **not run / unverified**, not passed.

“Zero failures” cannot be guaranteed for all environments. The standard is to execute the applicable checks, fix verified failures, and clearly report anything that remains unverified or blocked.

## 2. Required workflow

Follow this sequence for every meaningful change:

1. **Understand**
   - Read the relevant code, configuration, tests, architecture, and conventions.
   - Identify expected behavior, callers, dependencies, data flows, and acceptance criteria.
   - Inspect existing patterns before introducing a new one.
2. **Plan**
   - Break the work into minimal changes.
   - Identify security, compatibility, migration, performance, and testing risks.
   - Define acceptance criteria and validation commands.
3. **Implement**
   - Write the simplest maintainable solution that meets the requirements.
   - Preserve existing behavior unless a change is explicitly required.
   - Avoid unrelated edits and speculative abstractions.
4. **Test**
   - Add or update meaningful tests for changed behavior, including edge cases and regressions.
   - Run targeted tests first, then the relevant full suite.
5. **Analyze**
   - Run formatter, linter, compiler/type checker, dependency/security checks, and SonarQube analysis where available.
   - Fix issues at their root cause.
6. **Review**
   - Review the diff for correctness, security, readability, test quality, and unintended changes.
7. **Report**
   - Summarize changes, tests and commands actually executed, results, scan status, and remaining risks.
   - Distinguish verified results from assumptions and checks that could not be run.

## 3. General engineering standards

### Design and maintainability

- Apply SOLID where it improves design; do not force patterns without a concrete need.
- Follow DRY without creating premature abstractions. Duplication is sometimes preferable to a misleading abstraction.
- Keep functions, classes, modules, and components focused on one cohesive responsibility.
- Prefer explicit, readable control flow over clever or compressed code.
- Use meaningful names that communicate domain intent.
- Avoid magic numbers and strings; use named constants or configuration where appropriate.
- Keep business logic separate from transport, persistence, UI, and framework concerns.
- Favor composition over inheritance unless inheritance models a genuine substitutable relationship.
- Avoid unnecessary mutable state, global state, hidden side effects, and deep nesting.
- Remove dead code only when safe and relevant. Do not remove code simply because static analysis flags it without understanding its purpose.
- Comments should explain **why** or clarify non-obvious constraints, not restate the code.
- Do not add TODO/FIXME placeholders unless they are intentional, tracked, and explicitly disclosed.

### Error handling and observability

- Handle errors at the layer that can meaningfully recover or translate them.
- Never swallow exceptions or return success after a failed operation.
- Do not use exceptions for ordinary control flow.
- Preserve causal context when rethrowing or translating exceptions.
- Avoid leaking stack traces, secrets, internal details, or sensitive user data to clients.
- Log actionable context using structured logging where supported; redact sensitive fields.
- Do not log credentials, access tokens, passwords, payment details, or sensitive personal data.
- Avoid duplicate logging at multiple layers for the same failure.
- Use appropriate error codes and stable API error contracts.

### Performance

- Prefer algorithmic improvements over micro-optimizations.
- Avoid N+1 database access, unnecessary network calls, repeated expensive computation, and unbounded collections.
- Use pagination and limits for potentially large data sets.
- Consider timeouts, retries, backoff, cancellation, and resource cleanup for I/O.
- Do not add caching without a clear invalidation and consistency strategy.
- Do not sacrifice correctness or security for speculative performance gains.
- Benchmark or profile claims of performance improvement when feasible.

## 4. Java and Spring Boot standards

### Java language

- Follow the repository's Java version and formatting conventions.
- Use appropriate access modifiers; avoid unnecessarily public APIs.
- Prefer immutable data and `final` where practical.
- Use `try-with-resources` for `AutoCloseable` resources.
- Use `equals`/`hashCode` contracts correctly; do not compare objects with `==` unless identity is intended.
- Avoid raw types, unchecked casts, and unsafe generic usage.
- Use `Optional` primarily for return values where absence is meaningful; avoid using it for fields, parameters, or collections by default.
- Avoid broad `catch (Exception)` or `catch (Throwable)` unless at a justified boundary.
- Do not return `null` where a clear empty value, Optional, or explicit error contract is more appropriate.
- Avoid mutable static fields and shared mutable state.
- Use `BigDecimal` for monetary calculations; define scale and rounding explicitly.
- Ensure thread safety for shared state and concurrency-sensitive code.

### Spring Boot architecture

Use clear separation of responsibilities, adapting to the existing architecture:

- **Controller/API layer:** HTTP mapping, request validation, response mapping; no business logic or direct complex persistence logic.
- **Service/application layer:** business rules, orchestration, transaction boundaries.
- **Repository/data layer:** persistence access and query definitions.
- **DTOs:** API request/response contracts; do not expose persistence entities directly.
- **Configuration:** externalize environment-specific values; do not hardcode secrets or endpoints.
- **Mapping:** use explicit, maintainable mapping; avoid accidental data exposure.

### Spring-specific requirements

- Use constructor injection; avoid field injection.
- Apply `@Transactional` at appropriate service boundaries and understand propagation/isolation.
- Do not assume `@Transactional` works for self-invocation or private methods; understand proxy behavior.
- Validate request inputs using Bean Validation (`@Valid`, constraints) where appropriate.
- Use consistent exception translation and centralized handling (e.g. `@ControllerAdvice`) without exposing internal details.
- Avoid returning entities directly from controllers.
- Avoid over-fetching; use projections, pagination, and targeted queries where appropriate.
- Do not use `FetchType.EAGER` as a blanket fix for lazy-loading issues.
- Ensure database changes include safe, versioned migrations when the project uses migration tooling.
- Avoid blocking calls in reactive flows; do not mix reactive and imperative paradigms without a deliberate boundary.
- Configure timeouts for outbound HTTP calls and handle failure modes.
- Secure endpoints with least privilege and explicit authorization.

### Java testing

- Use the project's established test framework (typically JUnit 5, Mockito, Spring Boot Test).
- Unit-test business logic independently of Spring when possible.
- Use integration tests for persistence, framework wiring, serialization, security, and API behavior.
- Avoid mocking the unit under test or every dependency indiscriminately.
- Assert observable behavior, not implementation details.
- Include boundary conditions, invalid input, null/absent values where applicable, exceptions, and regression scenarios.
- Use deterministic tests; avoid sleeps, real external services, shared mutable fixtures, and order-dependent tests.
- Use Testcontainers or approved test infrastructure for realistic integration tests where appropriate.
- Do not use `@SpringBootTest` for every test; select the narrowest suitable test slice.
- Ensure tests clean up resources and do not rely on production data.

## 5. React and TypeScript standards

### React

- Use functional components and hooks consistent with project conventions.
- Keep components focused; extract reusable logic only when it improves clarity.
- Follow the Rules of Hooks.
- Keep rendering pure; avoid side effects during render.
- Use `useEffect` only to synchronize with external systems, not as a default place for derived state.
- Avoid unnecessary effects, duplicated state, and effect dependency workarounds.
- Use stable, meaningful keys for rendered lists; never use array indices as keys when list items can reorder or be inserted/deleted.
- Avoid prop drilling when it becomes a genuine maintainability problem; use context or state tools judiciously.
- Handle loading, empty, success, and error states for asynchronous UI.
- Prevent stale requests and race conditions; clean up subscriptions and abortable requests.
- Avoid unnecessary re-renders, but do not add memoization without a reason or evidence.
- Ensure forms provide validation, accessible labels, and understandable error feedback.

### TypeScript

- Enable and respect strict type checking where feasible.
- Avoid `any`; prefer `unknown` with safe narrowing when type information is uncertain.
- Avoid non-null assertions unless the invariant is proven and documented.
- Define precise interfaces/types for API boundaries and domain models.
- Validate untrusted runtime data; TypeScript types do not validate network responses.
- Avoid unsafe casts and duplicated type definitions.
- Use exhaustive handling for discriminated unions when practical.
- Keep types aligned with API contracts.

### Frontend security and accessibility

- Never render untrusted HTML without safe sanitization and a strong reason.
- Avoid putting secrets in frontend bundles or browser storage.
- Do not rely on client-side authorization; enforce access control on the server.
- Use semantic HTML, keyboard accessibility, visible focus, accessible names, and appropriate ARIA only when native semantics are insufficient.
- Avoid exposing sensitive data in URLs, browser logs, analytics, or error messages.

### Frontend testing

- Use the repository's established test stack (e.g. Vitest/Jest, React Testing Library).
- Test user-visible behavior and accessibility-relevant interactions.
- Cover validation, loading/error states, conditional rendering, and regressions.
- Avoid brittle snapshots as the only verification.
- Mock network boundaries, not internal implementation details.
- Run linting, type checking, unit tests, and production build.

## 6. Python and FastAPI standards

### Python

- Follow the project's supported Python version and formatter/linter configuration.
- Use type hints for public functions and important interfaces.
- Use clear names, small functions, and explicit control flow.
- Use context managers for resources.
- Avoid mutable default arguments.
- Avoid broad exception handling and bare `except`.
- Do not use `eval`/`exec` on untrusted input.
- Use dependency injection and explicit configuration rather than hidden global state.
- Keep I/O, domain logic, and API concerns separated.
- Use Decimal for financial values when precision is required.
- Avoid blocking I/O in async endpoints; use async-compatible clients or synchronous endpoints as appropriate.

### FastAPI

- Use Pydantic models for request/response validation and explicit response schemas.
- Keep endpoints thin; delegate business logic to service functions.
- Use `Depends` for shared dependencies and request-scoped resources.
- Use correct HTTP status codes and stable error response formats.
- Do not expose internal exception messages or tracebacks to clients.
- Configure CORS narrowly for known origins; never use permissive settings casually in production.
- Enforce authentication and authorization server-side, with least privilege.
- Use lifespan management for startup/shutdown resources where appropriate.
- Set timeouts and resource limits for external calls.
- Avoid blocking operations in `async def` endpoints.
- Ensure database sessions are correctly scoped and closed.
- Validate uploads, payload size, content type, and file handling where applicable.
- Keep OpenAPI documentation accurate and avoid exposing sensitive internal endpoints.

### Python testing

- Use the project's configured test framework (typically pytest).
- Use `TestClient` or the supported async client for API tests.
- Test validation failures, authentication/authorization, error mapping, and success paths.
- Use fixtures with clear scope and cleanup.
- Avoid test-order dependence, flaky timing, and production service dependencies.
- Run type checking (e.g. mypy/pyright) and lint/format checks if configured.

## 7. Security requirements

Apply security controls proportionate to the application and threat model:

- Validate and normalize untrusted input at system boundaries.
- Use parameterized queries/ORM APIs; never concatenate untrusted input into SQL.
- Prevent injection, path traversal, SSRF, XSS, CSRF, and insecure deserialization where relevant.
- Use secure password hashing; never store plaintext passwords.
- Use approved authentication and authorization mechanisms; verify issuer, audience, signature, expiry, and relevant claims for tokens.
- Apply least privilege to users, service accounts, API scopes, and database roles.
- Protect sensitive data in transit and at rest according to organizational policy.
- Avoid secrets in source code, committed configuration, test fixtures, and logs.
- Keep dependencies patched; investigate dependency and license policy findings.
- Do not disable TLS verification in production code.
- Use secure defaults and explicit allowlists where feasible.
- Follow applicable organizational policies and OWASP guidance.
- For security-sensitive changes, document threat assumptions and test the relevant negative cases.

## 8. Testing policy: no false green

The implementation is not complete merely because code compiles.

### Required test strategy

For each change, determine and apply relevant levels:

- **Unit tests:** isolated business rules and edge cases.
- **Integration tests:** framework, persistence, messaging, and service interactions.
- **API/contract tests:** request/response schemas, status codes, compatibility.
- **Regression tests:** reproduce the reported defect and prove it is fixed.
- **End-to-end tests:** critical user journeys where the project supports them.
- **Non-functional tests:** performance, security, accessibility, or resilience when relevant.

### Test quality rules

- Tests must be deterministic and independently runnable.
- Use realistic assertions; avoid asserting only that execution did not throw.
- Include negative and boundary cases.
- Do not hardcode expected values derived from the implementation itself.
- Do not weaken assertions, delete failing tests, skip tests, or reduce coverage solely to pass CI.
- If a test is legitimately obsolete, explain why and replace it with coverage for the intended behavior.
- Do not mark tests as ignored/skipped without a documented reason and approval where required.
- Fix flaky tests at their source; do not retry indefinitely and report the eventual pass as reliable.
- Preserve existing regression coverage.

### Test execution and reporting

Run the repository's actual commands. Typical examples (adapt to project files and versions):

```bash
# Java / Maven
./mvnw test
./mvnw verify

# Java / Gradle
./gradlew test
./gradlew check

# React / npm
npm ci
npm run lint --if-present
npm run typecheck --if-present
npm test -- --run
npm run build

# Python
python -m pytest
python -m ruff check .
python -m mypy .
```

Do not assume optional scripts exist. Inspect `package.json`, build files, and project configuration before running. Use the correct package manager and lockfile. Report the exact commands and outcomes.

## 9. SonarQube and static-analysis policy

### Quality gate

Treat the project's configured SonarQube/SonarCloud Quality Gate as authoritative for its configured rules and thresholds. A passing gate is necessary when required by the project, but it does not prove the code is defect-free or secure.

Before analysis:

- Confirm the correct project key, branch/PR context, scanner configuration, and server/CI setup.
- Confirm the scan includes the intended source and test files.
- Confirm coverage and test reports are generated and imported correctly.
- Do not expose tokens or credentials in command output or committed files.

### Finding remediation

For every relevant finding:

1. Read the complete rule description and affected code.
2. Determine whether it is a genuine defect, security risk, maintainability concern, or false positive.
3. Fix the underlying cause, not just the reported line.
4. Add or update tests when behavior or risk warrants it.
5. Re-run relevant tests and analysis.
6. Verify the finding is resolved in the new scan.
7. Record any unresolved item with its rule/key, rationale, owner/next action, and status.

### Do not game the scanner

- Do not add broad exclusions to hide source files or findings.
- Do not lower quality-gate thresholds to make a failing project pass.
- Do not add `NOSONAR`, suppressions, or issue ignores merely to silence warnings.
- Use a suppression only when the finding is demonstrably false positive or unavoidable, the rule permits it, and the project’s review/approval process accepts it.
- Keep suppressions narrow, justified, and documented with the reason and relevant ticket/reference.
- Do not alter scanner configuration or quality profiles without authorization.
- Do not confuse “no new issues” with “no existing issues.”
- Do not claim a Security Hotspot is resolved merely because it was reviewed; report its review status and disposition accurately.

### SonarQube checklist

- [ ] Scanner completed successfully.
- [ ] Correct branch/PR analyzed.
- [ ] Quality Gate status inspected.
- [ ] Bugs reviewed and fixed or explicitly dispositioned.
- [ ] Vulnerabilities reviewed and fixed or explicitly dispositioned.
- [ ] Code Smells reviewed and fixed where applicable.
- [ ] Security Hotspots reviewed according to policy.
- [ ] Coverage and duplication metrics are based on valid reports.
- [ ] No unjustified exclusions or suppressions were introduced.
- [ ] Remaining issues are disclosed; no false “all clear” claim.

Typical Maven invocation (adapt to the organization’s configured scanner and credentials; never put tokens in source control):

```bash
./mvnw verify sonar:sonar
```

A successful scanner execution is **not** equivalent to a passing Quality Gate. Retrieve and inspect the gate result through the configured CI/SonarQube workflow.

## 10. CI/CD and dependency hygiene

- Keep builds reproducible using committed lockfiles and pinned/managed versions.
- Do not upgrade dependencies without checking compatibility and security impact.
- Avoid unmaintained or unnecessary dependencies.
- Run build, tests, static analysis, and security checks in CI as required by the repository.
- Fail the pipeline on required checks that fail; do not bypass checks to merge.
- Never commit generated secrets, local environment files, credentials, or build artifacts unless explicitly required.
- Keep CI permissions least-privileged and protect deployment credentials.
- Verify migrations and deployment changes are backward-compatible where needed.
- Do not claim a pipeline passed unless its actual run is inspected.

## 11. Pull request and code review checklist

Before proposing a change, verify:

### Correctness
- [ ] Acceptance criteria are met.
- [ ] Existing behavior and API contracts are preserved unless intentionally changed.
- [ ] Edge cases and error paths are addressed.
- [ ] No race conditions or resource leaks are introduced.

### Design and maintainability
- [ ] Responsibilities and dependencies are clear.
- [ ] Names and abstractions are meaningful.
- [ ] No unnecessary duplication, complexity, or dead code.
- [ ] No unrelated changes are included.

### Security and privacy
- [ ] Inputs are validated and authorization enforced.
- [ ] No secrets or sensitive data are exposed.
- [ ] Dependencies and external calls are handled safely.
- [ ] Logging and error messages are safe.

### Validation
- [ ] Relevant tests were added/updated.
- [ ] Tests and build were actually run.
- [ ] Lint, type checking, and static analysis were run when available.
- [ ] SonarQube status is accurately reported.
- [ ] Any skipped/unavailable checks are disclosed.

## 12. Definition of Done

A task may be reported as **Done** only when all applicable items below are satisfied:

1. Requirements and acceptance criteria are implemented.
2. Code follows the repository's conventions and this skill.
3. Tests cover the changed behavior and meaningful edge cases.
4. Relevant tests pass in the actual execution environment.
5. Build, lint, and type checks pass where configured and applicable.
6. SonarQube analysis has completed and the required Quality Gate is verified, if available/required.
7. Relevant security and dependency checks have been completed, if available/required.
8. The diff has been reviewed for regressions, secrets, and unrelated changes.
9. Documentation/configuration/migrations are updated where needed.
10. Results are reported truthfully with exact checks and any limitations.

If any required check fails, the status must be **Blocked**, **Failed**, or **Partially verified**—not Done. If a check cannot be run, state why and identify what is needed to verify it.

## 13. Required final response format for coding tasks

Use this concise structure:

```text
Implemented
- What changed and why.

Validation
- Command: <actual command> — PASS/FAIL/NOT RUN
- Tests: <actual result>
- Build/lint/type checks: <actual result>
- SonarQube: <Quality Gate status, scan date/branch if known, or NOT RUN>
- Security checks: <actual result or NOT RUN>

Remaining risks / follow-up
- List unresolved findings, limitations, or required manual checks.
- If none are known, say “No known outstanding issues from the checks performed.”
```

Do not state “all SonarQube issues passed,” “zero bugs,” “100% test coverage,” or “production-ready” without evidence supporting the exact claim.

## 14. Project-specific configuration takes precedence

This skill establishes default standards. Follow stricter organizational, regulatory, repository, framework, and security policies when they apply. Do not silently weaken those policies.

When standards conflict:
1. Security and legal requirements take precedence.
2. Explicit repository/organization standards take precedence over generic guidance.
3. Preserve existing project conventions unless there is a justified reason to change.
4. Explain material conflicts and request a decision when necessary.

## 15. Agent operating instruction

Act as a careful senior engineer and reviewer. Make the smallest correct change, validate it honestly, and leave the codebase easier to understand and maintain. Never trade correctness, security, or test integrity for a green checkmark.
