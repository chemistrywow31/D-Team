---
name: CSO Audit
description: Comprehensive security audit covering 12 phases from secrets archaeology to STRIDE threat modeling
---

# CSO Audit

Perform a comprehensive security audit. Think like an attacker, report like a defender. Every finding must include a concrete exploit scenario — no theoretical risks.

## Invocation Modes

- `/cso-audit` — Full audit with 8/10 confidence threshold (zero noise)
- `/cso-audit --comprehensive` — Deep scan with 2/10 threshold (surfaces tentative findings marked `[TENTATIVE]`)
- `/cso-audit --diff` — Branch-only analysis (scope to current diff)
- `/cso-audit --owasp` — OWASP Top 10 focused scan only
- `/cso-audit --supply-chain` — Dependencies and CI/CD only

## 12-Phase Methodology

### Phase 1: Architecture Detection

Identify the tech stack, frameworks, and deployment model. Build a mental model of the system's attack surface before scanning.

### Phase 2: Secrets Archaeology

- Scan git history for leaked credentials: `git log -p --all -S 'password' -S 'secret' -S 'api_key' -S 'token'`
- Check for tracked `.env` files, credential files, or inline secrets
- Verify CI/CD secrets use `${{ secrets.X }}`, not hardcoded values
- Validate secret format (correct length, prefix) to confirm they are real

### Phase 3: Supply Chain Audit

- Check dependencies for known vulnerabilities (CVE databases)
- Inspect install scripts in dependencies for suspicious behavior
- Verify lockfile integrity (no tampered checksums)
- Confirm vulnerable functions are actually imported/called before flagging

### Phase 4: CI/CD Security

- Audit GitHub Actions / CI configs for unpinned actions
- Check for `pull_request_target` risks (code execution from forks)
- Verify no script injection via untrusted input in CI commands
- Check artifact handling and publish step security

### Phase 5: Infrastructure Hardening

- Dockerfile: non-root user, pinned base images, multi-stage builds, no secrets in layers
- IaC configs: no hardcoded credentials, least-privilege policies
- Database: connection encryption, access control, backup security

### Phase 6: Webhook & Integration Audit

- Verify webhook signature verification in middleware
- Check TLS validation on outbound connections
- Audit OAuth scope requests (minimum necessary scopes)
- Verify API key rotation capability

### Phase 7: LLM & AI Security

- Trace user input flow to system prompt construction (prompt injection vectors)
- Check for unsanitized LLM output rendered in UI (XSS via LLM)
- Verify tool call validation (LLM cannot invoke unauthorized tools)
- Check for cost amplification vectors (unbounded loops, recursive calls)

### Phase 8: OWASP Top 10

| Category | Check |
|----------|-------|
| A01 Broken Access Control | Missing auth checks, IDOR, privilege escalation |
| A02 Cryptographic Failures | Weak hashing, plaintext storage, missing TLS |
| A03 Injection | SQL, XSS, command, LDAP, template injection |
| A04 Insecure Design | Missing rate limits, no abuse controls |
| A05 Security Misconfiguration | Debug mode, default credentials, verbose errors |
| A06 Vulnerable Components | Known CVEs in dependencies |
| A07 Auth Failures | Weak passwords, missing MFA, session fixation |
| A08 Integrity Failures | Missing code signing, unsafe deserialization |
| A09 Logging Failures | Missing audit logs, PII in logs |
| A10 SSRF | Unvalidated URLs in server-side requests |

### Phase 9: STRIDE Threat Model

For each major component, assess:
- **S**poofing — Can identity be faked?
- **T**ampering — Can data be modified in transit/storage?
- **R**epudiation — Can actions be denied without audit trail?
- **I**nformation Disclosure — Can sensitive data leak?
- **D**enial of Service — Can the component be overwhelmed?
- **E**levation of Privilege — Can permissions be escalated?

### Phase 10: Data Classification

Categorize data by risk level: Restricted (credentials, PII), Confidential (business data), Internal (logs, metrics), Public. Verify handling matches classification.

### Phase 11: False Positive Filtering

- Trace every finding to active code paths — do not flag dead code
- Verify confidence meets the threshold (8/10 for daily, 2/10 for comprehensive)
- Run variant analysis: if one instance found, search for similar patterns

### Phase 12: Report Generation

Produce the structured findings report.

## Report Format

```
## CSO Security Audit Report

**Date**: [YYYY-MM-DD]
**Mode**: [Daily | Comprehensive | Diff | Scoped]
**Stack**: [Detected tech stack]

### Executive Summary
- Critical: N | High: N | Medium: N | Low: N
- Verdict: [PASS | CONDITIONAL PASS | BLOCK]

### Findings

#### [CRITICAL-001] [Title]
- **Category**: [OWASP/STRIDE category]
- **File**: [path:line]
- **Exploit Scenario**: [Concrete attack description]
- **Impact**: [What an attacker gains]
- **Remediation**: [Specific fix with code example]
- **Confidence**: [N/10]

### Remediation Priority
1. [Highest impact, lowest effort fixes first]

### Disclaimer
This audit is AI-assisted and not a substitute for professional security review.
```

## Example

Input: `/cso-audit --diff`

Output: Scans current branch diff for security issues. Finds: unparameterized SQL query in new endpoint (CRITICAL), missing rate limit on new auth route (HIGH), debug logging exposing user email (MEDIUM). Each finding includes file:line, exploit scenario, and specific fix.

## Guardrails

- Never modify code — audit only, produce findings and recommendations
- Never make live requests to test vulnerabilities — code tracing only
- Every finding must include a concrete exploit scenario, not just "this is risky"
- Hard exclusions: DoS theoretical claims, missing hardening (not concrete vulnerabilities), test fixtures
- Respect confidence thresholds — do not report below the mode's gate
