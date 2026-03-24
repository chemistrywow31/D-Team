---
name: Review Checklist
description: Structured checklist-driven code review with test coverage audit and scope verification
---

# Review Checklist

Perform a structured, checklist-driven code review. This skill supplements the code-reviewer agent's fix-first flow with a systematic coverage guarantee — every category is checked, nothing is skipped by oversight.

## Workflow

### Step 1: Scope Detection

```bash
git diff origin/main --stat
git diff origin/main
```

1. Read the full diff before flagging any issues
2. Identify the stated intent (PR description, task description, or commit messages)
3. Verify the diff delivers what was stated — flag `[SCOPE-DRIFT]` for anything extra

### Step 2: Two-Pass Checklist Review

Run through every category below. Skip categories where no issues are found. Cite `file:line` for every finding.

### Step 3: Test Coverage Audit

For every new code path in the diff:
1. Trace each branch, function, and user flow
2. Map against existing tests — does a test exercise this path?
3. Flag untested paths with priority:
   - CRITICAL: Security paths, auth, payment, data mutation
   - HIGH: Core business logic branches
   - MEDIUM: Error handling, edge cases
   - LOW: Cosmetic, logging

Auto-generate simple unit tests where safe. Ask for E2E and integration tests.

### Step 4: Fix-First Classification

Classify every finding as AUTO-FIX or ASK per the fix-first heuristic, then apply.

## Review Checklist

### Pass 1 — CRITICAL

#### SQL & Data Safety
- String interpolation in SQL → use parameterized queries
- TOCTOU races: check-then-set patterns that must be atomic
- Bypassing model validations for direct DB writes
- N+1 queries: missing eager loading for associations used in loops

#### Race Conditions & Concurrency
- Read-check-write without uniqueness constraint or duplicate key handling
- Find-or-create without unique DB index
- Status transitions without atomic WHERE-UPDATE
- Unsafe HTML rendering on user-controlled data (XSS)

#### LLM Output Trust Boundary
- LLM-generated values persisted without format validation
- Structured tool output accepted without type/shape checks

#### Enum & Value Completeness
When the diff adds a new enum value, status string, or type constant:
1. Trace it through every consumer (read code, not just grep)
2. Check allowlists/filter arrays for missing inclusion
3. Check case/if-elsif chains for unhandled new value

### Pass 2 — INFORMATIONAL

#### Conditional Side Effects
- Code paths that branch but forget a side effect on one branch
- Log messages claiming an action happened when conditionally skipped

#### Magic Numbers & String Coupling
- Bare numeric literals used across files → named constants
- Error message strings used as query filters elsewhere

#### Dead Code & Consistency
- Variables assigned but never read
- Version mismatch between PR title and VERSION/CHANGELOG
- Comments describing old behavior after code changed

#### Test Gaps
- Negative-path tests missing side effect assertions
- Security enforcement features without integration tests
- Missing `.expects(:something).never` for paths that must NOT call external services

#### Completeness Gaps
- Shortcut implementations where the 100% version costs <30 minutes
- Features at 80-90% when 100% is achievable with modest additional code

#### Crypto & Entropy
- Truncation instead of hashing (less entropy, collision risk)
- `rand()` for security-sensitive values → use secure random
- Non-constant-time comparisons on secrets (timing attack)

#### Performance & Bundle Impact
- Known-heavy dependencies added (moment.js, full lodash, jquery)
- Images without `loading="lazy"` or explicit dimensions
- Large static assets (>500KB) committed to repo
- `useEffect` fetch waterfalls (combine or parallelize)

Do NOT flag: devDependencies, dynamic imports, small utilities (<5KB gzipped).

## Fix-First Heuristic

```
AUTO-FIX (apply immediately):          ASK (needs human judgment):
├─ Dead code / unused variables         ├─ Security (auth, XSS, injection)
├─ N+1 queries (clear fix pattern)      ├─ Race conditions
├─ Stale comments contradicting code    ├─ Design decisions
├─ Magic numbers → named constants      ├─ Large fixes (>20 lines)
├─ Missing LLM output validation        ├─ Enum completeness
├─ Version/path mismatches              ├─ Removing functionality
└─ Inline styles, O(n*m) view lookups   └─ User-visible behavior changes
```

Critical findings default toward ASK. Informational findings default toward AUTO-FIX.

## Suppressions — Do NOT Flag

- Redundancy that aids readability
- "Add a comment explaining this threshold" — thresholds change, comments rot
- Consistency-only changes with no functional impact
- Eval threshold changes (tuned empirically)
- Anything already addressed in the diff

## Output Format

```
## Checklist Review: [description]

### Scope Check
- Intent: [stated scope]
- Drift: [none | list]

### Test Coverage Audit
- New paths: N
- Tested: N
- Untested: N (CRITICAL: X, HIGH: Y, MEDIUM: Z)
- Auto-generated tests: [list if any]

### Findings (Pass 1 — CRITICAL)
- [category] [file:line] — [issue] → [AUTO-FIX applied | ASK: options]

### Findings (Pass 2 — INFORMATIONAL)
- [category] [file:line] — [issue] → [AUTO-FIX applied | ASK: options]

### Summary
AUTO-FIXED: N | ASK: N | CRITICAL: N | INFORMATIONAL: N
```

## Example

Input: `/review-checklist`

Output: Reads diff, runs two-pass checklist. Finds: N+1 query in new endpoint (AUTO-FIXED with eager loading), new `status` enum value not handled in billing switch statement (ASK: add case or confirm intentional fallthrough), 3 untested error paths in payment flow (generates unit tests). Summary: 1 auto-fixed, 1 judgment call, 3 test gaps filled.
