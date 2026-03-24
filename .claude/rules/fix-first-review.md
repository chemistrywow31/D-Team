---
name: Fix-First Code Review
description: Code reviewer auto-fixes mechanical issues and escalates judgment calls to the user
---

# Fix-First Code Review

## Applicability

- Applies to: `code-reviewer`
- Scope: All code review activities

## Rule Content

### Fix-First Principle

When reviewing code, separate issues into two categories and handle them differently:

1. **Mechanical issues** — Auto-fix immediately with `[AUTO-FIXED]` attribution
2. **Judgment calls** — Surface to the user for decision

This separation maintains development velocity by eliminating round-trips for obvious fixes while preserving human oversight for decisions that require context.

### Auto-Fix Category (Mechanical Issues)

Fix these issues automatically without asking the user. Tag each fix with `[AUTO-FIXED]` in your review output:

- Dead code removal (unused imports, unreachable branches, commented-out code)
- Stale comments that no longer match the code
- Magic numbers replacement with named constants
- N+1 query detection and fix (when the fix pattern is clear)
- Consistent formatting violations (indentation, trailing whitespace, missing semicolons)
- Missing error context (bare `return err` → wrapped error with context)
- Typos in variable names, comments, or strings
- Redundant type assertions or unnecessary type casts
- Debug statements left in production code (`console.log`, `fmt.Println`, `print()`)

After auto-fixing, report:
```
[AUTO-FIXED] Removed 3 unused imports in api/handler.go
[AUTO-FIXED] Replaced magic number 86400 with SECONDS_PER_DAY constant
[AUTO-FIXED] Added error context to 2 bare return statements
```

### Ask-User Category (Judgment Calls)

Surface these issues to the user for decision. Do not auto-fix:

- **Security concerns**: Authentication logic, authorization checks, input sanitization, encryption choices
- **Race conditions**: Concurrent access patterns, lock ordering, atomic operations
- **Design decisions**: API contract changes, data model modifications, architectural patterns
- **Performance trade-offs**: Caching strategies, query optimization approaches, algorithm choices
- **Business logic**: Validation rules, error handling behavior, edge case treatment
- **Breaking changes**: Public API modifications, database schema changes, config format changes

Present each judgment call with:
```
[ASK] Potential race condition in user session handling
File: auth/session.go:45
Issue: Two goroutines may access session map concurrently without locking.
Options:
  A) Add sync.RWMutex (safe, minimal change)
  B) Switch to sync.Map (better for read-heavy, larger change)
  C) Accept risk (document as known limitation)
Recommendation: Option A
```

### 30-Minute Threshold

Only flag a gap or suggest an improvement if the 100% solution costs less than 30 minutes of agent time. If a proper fix would take longer:

- Note the issue as `[TECH-DEBT]` in the review
- Do not block the merge
- Recommend creating a follow-up task

This prevents reviews from ballooning into refactoring sessions.

### Test Coverage Audit (During Review)

Every review must include a test coverage audit for new code:

1. Trace every new branch, function, and user flow in the diff
2. Map each path to an existing test
3. Flag untested paths by priority:
   - CRITICAL: Security paths, auth, payment, data mutation
   - HIGH: Core business logic branches
   - MEDIUM: Error handling, edge cases
   - LOW: Cosmetic, logging
4. Auto-generate simple unit tests where safe (tag `[AUTO-TEST]`)
5. Ask user for E2E and integration test decisions

Include in review output:
```
### Test Coverage Audit
- New paths: N
- Tested: N
- Untested: N (CRITICAL: X, HIGH: Y, MEDIUM: Z)
- [AUTO-TEST] Generated unit test for validateInput() in tests/validate.test.ts
```

### Review Checklist Integration

Use the `/review-checklist` skill for structured, category-by-category coverage. The checklist ensures no review category is skipped by oversight. Combine checklist findings with the fix-first classification below.

### Two-Pass Review Structure

Execute reviews in two passes:

**Pass 1 — CRITICAL (must fix before merge):**
- SQL injection, XSS, command injection
- Race conditions in shared state
- Unhandled LLM/API output used in security context
- Missing enum/switch exhaustive handling
- Authentication/authorization bypasses

**Pass 2 — INFORMATIONAL (fix or acknowledge):**
- Side effects in functions that appear pure
- Magic numbers and unnamed constants
- Dead code and stale comments
- Missing test coverage for new code
- N+1 queries

### Scope Drift Detection

Before reviewing code quality, verify that the delivered code matches the stated intent:

1. Read the PR description or task description
2. Compare the actual changes against the stated scope
3. Flag any changes that fall outside the stated scope as `[SCOPE-DRIFT]`

Scope drift is a WARNING, not a BLOCK — but it must be called out.

### Review Output Format

```
## Code Review: [PR/Change description]

### Scope Check
- Intent: [what the change claims to do]
- Drift: [none | list of out-of-scope changes]

### Auto-Fixed
- [AUTO-FIXED] [description]
- [AUTO-FIXED] [description]

### Judgment Calls
- [ASK] [title] — [brief description with options]
- [ASK] [title] — [brief description with options]

### Issues
- [CRITICAL] [description with file:line]
- [WARNING] [description with file:line]
- [TECH-DEBT] [description, estimated effort]

### Verdict: [APPROVE | WARNING | BLOCK]
Files Reviewed: [count]
Issues: CRITICAL: N, WARNING: N, AUTO-FIXED: N, ASK: N, TECH-DEBT: N
```

## Violation Determination

- Auto-fixing a judgment call (security, race condition, design decision) without asking the user → Violation
- Surfacing a mechanical issue as a judgment call when it has a clear, safe fix → Violation (wastes user time)
- Blocking a merge for an issue that exceeds the 30-minute threshold without marking as TECH-DEBT → Violation
- Skipping the scope drift check → Violation
- Review output missing the structured format (no verdict, no categorization) → Violation

## Exceptions

- When the user explicitly says "fix everything you find", all mechanical issues may be auto-fixed without individual attribution
- First-time setup of a new module or feature may skip scope drift detection (there is no prior scope to drift from)
