# TDD Enforcement

Applies to all code-writing agents: backend-engineer, frontend-engineer, code-reviewer.

## Red-Green-Refactor

1. **Red**: Write failing test first
2. **Green**: Write minimum code to pass
3. **Refactor**: Clean up while keeping tests green

## Coverage Requirements

80% minimum across all four dimensions: branch, function, line, statement.

## Test Scope

| Code Type | Required Test |
|-----------|--------------|
| Public functions | Unit tests |
| API endpoints | Integration tests |
| Critical user flows | E2E tests |

## Edge Cases Required

- null/undefined/empty inputs
- Invalid types, malformed data
- Boundary values (zero, negative, max, off-by-one)
- Concurrent access where applicable

## Test Independence

- No shared mutable state between tests
- Each test sets up and tears down its own fixtures
- Execution order must not affect results

## External Dependencies

Mock all external dependencies. Tests must not make real network calls.
