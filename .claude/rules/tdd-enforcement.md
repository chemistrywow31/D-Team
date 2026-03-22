---
name: TDD Enforcement
description: Enforce test-driven development practices and minimum coverage standards for all code-writing agents
---

# TDD Enforcement

## Applicability

- Applies to: All code-writing agents (`backend-engineer`, `frontend-engineer`, `code-reviewer`)

## Rule Content

### Red-Green-Refactor Cycle

You must write failing tests before writing implementation code. Follow this sequence strictly:

1. **Red**: Write a test that defines expected behavior. Run it to confirm it fails.
2. **Green**: Write the minimum implementation code to make the test pass.
3. **Refactor**: Clean up the implementation while keeping all tests green.

You must not submit implementation code without corresponding tests written first.

### Coverage Requirements

You must meet a minimum of 80% test coverage across all four dimensions:

- **Branch coverage**: 80% or higher
- **Function coverage**: 80% or higher
- **Line coverage**: 80% or higher
- **Statement coverage**: 80% or higher

Run coverage verification before marking any task complete. Use the test coverage command appropriate for your project's language and framework.

### Test Scope by Code Type

| Code Type | Required Test Type |
|---|---|
| Public functions | Unit tests |
| API endpoints | Integration tests |
| Critical user flows | End-to-end (E2E) tests |

You must write unit tests for every public function. You must write integration tests for every API endpoint. You must write E2E tests for every critical user flow identified in the requirements.

### Edge Case Coverage

You must include test cases for the following edge conditions:

- `null` and `undefined` inputs (or language equivalent)
- Empty strings, empty arrays, empty objects
- Invalid input types and malformed data
- Boundary values (zero, negative, max int, min int, off-by-one)
- Concurrent access scenarios where applicable

### Test Independence

You must ensure every test runs independently. Tests must not share mutable state. Each test must set up its own fixtures and tear them down after execution. Test execution order must not affect results.

### External Dependency Mocking

You must mock all external dependencies (databases, APIs, file systems, third-party services). Tests must not make real network calls or depend on external service availability.

## Violation Determination

- Submitting implementation code without a corresponding test written before it → Violation
- Test coverage below 80% on any of the four dimensions (branch, function, line, statement) → Violation
- A public function exists without a unit test → Violation
- An API endpoint exists without an integration test → Violation
- Tests share mutable state or depend on execution order → Violation
- Tests make real network calls to external services instead of using mocks → Violation
- Edge cases (null, empty, invalid, boundary) are not covered in test suites → Violation

## Exceptions

- Prototype or spike code explicitly marked as `// SPIKE: <reason>` is exempt from coverage requirements but must be replaced with TDD-compliant code before merging to main
