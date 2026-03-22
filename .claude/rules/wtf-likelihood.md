---
name: WTF-Likelihood Stop-Loss
description: Prevent execution agents from runaway auto-fixing by enforcing risk thresholds and escalation gates
---

# WTF-Likelihood Stop-Loss

## Applicability

- Applies to: All execution agents (`backend-engineer`, `frontend-engineer`, `devops-engineer`, `build-resolver`, `e2e-runner`, `refactor-cleaner`)
- Scope: Any automated fixing, refactoring, or iterative problem-solving

## Rule Content

### WTF-Likelihood Assessment

Before applying any fix, assess the WTF-likelihood: the probability that this fix introduces a new bug, breaks existing behavior, or makes the codebase harder to understand.

WTF-likelihood factors:
- **Unfamiliarity**: You have not read the surrounding code thoroughly
- **Side effects**: The change touches shared state, global config, or widely-imported modules
- **Complexity**: The fix requires changes across multiple files or layers
- **Uncertainty**: You are guessing at the root cause rather than confirming it
- **Pattern violation**: The fix deviates from established patterns in the codebase

### Risk Threshold: 20%

If your assessed WTF-likelihood exceeds 20%, you must STOP and escalate to the user via the AskUserQuestion tool. Present:

1. What you are trying to fix
2. Your proposed fix
3. Why the risk is elevated (which factors above apply)
4. Alternative approaches you considered

Do not apply the fix without user approval when risk exceeds the threshold.

### Hard Limit: 50 Fixes Per Session

You must not apply more than 50 fixes in a single session. After reaching 50 fixes, stop and report progress to the coordinator. This prevents cascading fix chains where each fix triggers more fixes.

Track your fix count. When approaching the limit (40+ fixes), proactively summarize progress and ask whether to continue in a new session.

### 3-Strike Rule for Failed Hypotheses

When investigating a bug or failure:

1. Form a hypothesis about the root cause
2. Implement a targeted fix based on that hypothesis
3. Verify the fix resolves the issue

If the fix does not resolve the issue, that is one strike. After 3 failed hypotheses:

- STOP attempting fixes
- Use the AskUserQuestion tool to present:
  - The 3 hypotheses you tried and why each failed
  - Your current best understanding of the root cause
  - Whether to continue investigating, escalate, or add logging/instrumentation

Do not continue guessing past 3 strikes.

### Blast Radius Check

Before applying any fix, assess the blast radius — how many files and modules the change touches.

| Blast Radius | Action |
|--------------|--------|
| 1-2 files in same module | Proceed |
| 3-5 files in same module | Proceed with caution, verify tests pass |
| >5 files | STOP — confirm with user via AskUserQuestion before proceeding |
| Cross-module changes | STOP — confirm with user via AskUserQuestion before proceeding |

### Scope Lock

After forming a hypothesis about a bug's root cause, restrict your edits to the affected directory or module. Do not expand the scope of changes beyond what the hypothesis requires.

If you discover the root cause is in a different module than expected, STOP, update your hypothesis, reassess WTF-likelihood, and proceed only if the new assessment is below 20%.

### No Verification = No Ship

You must never apply a fix you cannot verify. Every fix must be validated by at least one of:

- Running the relevant test suite and confirming the failure is resolved
- Running the application and confirming the behavior is correct
- Running a type check / lint / build and confirming the error is resolved

If you cannot verify a fix (no tests, no build command, no way to check), report this to the coordinator and recommend adding verification infrastructure before proceeding.

## Violation Determination

- Applying a fix with assessed WTF-likelihood >20% without user approval → Violation
- Exceeding 50 fixes in a single session without stopping → Violation
- Continuing to guess after 3 failed hypotheses without escalating → Violation
- Applying a fix touching >5 files without user confirmation → Violation
- Applying a fix outside the scope-locked directory without reassessment → Violation
- Applying a fix with no verification step → Violation

## Exceptions

- Emergency hotfixes explicitly authorized by the user may bypass the 3-strike rule, but the risk threshold and blast radius checks still apply
- Build resolver may exceed the 5-file blast radius when applying a single coherent fix (e.g., updating an import path across many files) if the change is mechanical and verifiable
