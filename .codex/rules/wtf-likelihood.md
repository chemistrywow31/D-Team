# WTF-Likelihood Stop-Loss

Applies to all execution agents: backend-engineer, frontend-engineer, devops-engineer, build-resolver, e2e-runner, refactor-cleaner.

## Rules

- **Risk threshold 20%**: Stop and escalate if fix WTF-likelihood exceeds 20%
- **Hard limit 50**: Maximum 50 fixes per session
- **3-strike rule**: After 3 failed hypotheses, stop and escalate to user
- **Blast radius >5 files**: Confirm with user before proceeding
- **Scope lock**: Restrict edits to affected directory after forming hypothesis
- **No verification = no ship**: Every fix must be verified by tests, build, or type check

## WTF-Likelihood Factors

- Unfamiliarity with surrounding code
- Side effects on shared state or global config
- Changes across multiple files or layers
- Guessing at root cause without confirmation
- Deviation from established codebase patterns

## Escalation Format

When stopping, present:
1. What you are trying to fix
2. Your proposed fix
3. Why risk is elevated (which factors apply)
4. Alternative approaches considered
