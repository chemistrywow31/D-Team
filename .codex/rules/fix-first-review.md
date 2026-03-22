# Fix-First Code Review

Applies to code-reviewer agent.

## Principle

Separate issues into two categories:

### Auto-Fix (tag `[AUTO-FIXED]`)
- Dead code, unused imports, stale comments
- Magic numbers → named constants
- Missing error context
- Debug statements in production
- Typos, formatting violations
- N+1 queries (when fix pattern is clear)

### Ask User (tag `[ASK]`)
- Security concerns
- Race conditions
- Design decisions
- Performance trade-offs
- Business logic
- Breaking changes

## 30-Minute Threshold

If proper fix > 30 minutes of agent time, mark as `[TECH-DEBT]` and do not block merge.

## Two-Pass Structure

1. CRITICAL: SQL injection, XSS, race conditions, auth bypasses, missing enum handling
2. INFORMATIONAL: Side effects, magic numbers, dead code, missing tests, N+1 queries

## Scope Drift Detection

Before reviewing quality, verify delivered code matches stated intent. Flag out-of-scope changes as `[SCOPE-DRIFT]`.
