---
name: Investigate
description: Systematic root-cause debugging through hypothesis-driven investigation with 3-strike escalation
---

# Investigate

Perform systematic root-cause debugging. Never fix without confirming root cause first.

## The Stance

- **Hypothesis-driven** — Form a theory, test it, confirm or reject
- **Evidence-based** — Every conclusion must cite code, logs, or reproduction output
- **Minimal-touch** — Fix only the root cause; do not "improve" surrounding code
- **WTF-likelihood aware** — Assess risk before every fix attempt

## 5-Phase Workflow

### Phase 1: Symptom Collection

1. Gather all available information about the bug:
   - Error messages, stack traces, log output
   - Steps to reproduce
   - When it started (check `git log` for recent changes)
   - Who/what is affected (scope assessment)
2. Read the code paths involved — do not guess from error messages alone
3. Check if the issue is reproducible locally

Output: Clear symptom summary with reproduction steps.

### Phase 2: Pattern Analysis

Match symptoms against known bug patterns:

| Pattern | Indicators |
|---------|-----------|
| Race condition | Intermittent failures, timing-dependent, works in debug mode |
| Nil/null propagation | NilClass error, undefined is not a function, segfault |
| State corruption | Correct on first call, wrong on subsequent calls |
| Off-by-one | Works for most inputs, fails at boundaries |
| Environment drift | Works locally, fails in CI/staging/production |
| Dependency conflict | Broke after upgrade, works with pinned version |
| Data migration | Existing records fail, new records work |

Read the actual code — do not rely solely on pattern matching.

### Phase 3: Hypothesis Formation and Testing

1. Form a specific, falsifiable hypothesis: "The bug occurs because X happens when Y"
2. Identify the minimal test to confirm or reject the hypothesis
3. Test it:
   - Add temporary logging or assertions
   - Write a failing test that reproduces the bug
   - Use debugger breakpoints if available
4. If confirmed → proceed to Phase 4
5. If rejected → count as one strike, form next hypothesis

**3-Strike Rule**: After 3 rejected hypotheses, STOP. Present:
- The 3 hypotheses tested and evidence for rejection
- Current best understanding of the root cause
- Recommended next step (more instrumentation, pair debugging, or escalate)

### Phase 4: Root Cause Fix

1. Write a regression test that fails with the current bug
2. Apply the minimal fix targeting the confirmed root cause
3. Verify the regression test passes
4. Run the full test suite to check for regressions
5. Create an atomic git commit with the fix

**Fix scope rules:**
- Fix only what the hypothesis identified — nothing else
- If the fix touches >5 files, stop and confirm with user
- If WTF-likelihood >20%, stop and present alternatives

### Phase 5: Verification

1. Reproduce the original bug scenario — confirm it no longer occurs
2. Run the full test suite — confirm no regressions
3. If the bug was environment-specific, verify in that environment
4. Document the root cause in the commit message

## Output Format

```
## Investigation Report

### Symptom
[What was observed]

### Root Cause
[Confirmed cause with code reference]

### Hypotheses Tested
1. [Hypothesis] → [Confirmed/Rejected] — [Evidence]
2. [Hypothesis] → [Confirmed/Rejected] — [Evidence]

### Fix Applied
- File: [path:line]
- Change: [Description of fix]
- Regression test: [test file path]

### Verification
- Original scenario: [PASS/FAIL]
- Full test suite: [PASS/FAIL]
- Strikes used: N/3
```

## Example

Input: `/investigate "Users intermittently get 500 error on /api/checkout"`

Output:
1. Collect symptoms: 500 errors in logs, happens ~5% of requests, no pattern by user
2. Pattern match: Intermittent → suspect race condition or external dependency timeout
3. Hypothesis 1: "Payment gateway timeout not handled" → Read checkout handler → Confirmed: `await gateway.charge()` has no timeout, default is 30s, server timeout is 25s
4. Fix: Add 20s timeout to gateway call with proper error handling
5. Verify: Regression test with mocked timeout passes, full suite green

## Guardrails

- Never fix without a confirmed hypothesis — guessing wastes time and risks regressions
- Never expand scope beyond the root cause — file a separate issue for related problems
- Always write a regression test before applying the fix
- Always run full test suite after the fix
- Respect WTF-likelihood thresholds at all times
