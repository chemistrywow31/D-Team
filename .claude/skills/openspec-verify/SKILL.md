---
name: OpenSpec Verify Change
description: Verify implementation matches change artifacts across completeness, correctness, and coherence dimensions
---

# OpenSpec Verify Change

Verify that an implementation matches the change artifacts (specs, tasks, design).

## Input

Optionally specify a change name. If omitted, prompt for selection from available changes.

## Steps

1. **If no change name provided, prompt for selection**

   Run `openspec list --json` to get available changes. Use AskUserQuestion to let the user select.

2. **Load change artifacts**

   Read all available artifacts from the change directory:
   - `openspec/changes/<name>/proposal.md`
   - `openspec/changes/<name>/specs/`
   - `openspec/changes/<name>/design.md`
   - `openspec/changes/<name>/tasks.md`

3. **Verify Completeness**

   **Task Completion:**
   - Parse checkboxes in tasks.md: `- [ ]` (incomplete) vs `- [x]` (complete)
   - Flag incomplete tasks as CRITICAL

   **Spec Coverage:**
   - Extract all requirements from delta specs
   - Search codebase for implementation evidence
   - Flag unimplemented requirements as CRITICAL

4. **Verify Correctness**

   **Requirement-Implementation Mapping:**
   - For each requirement, find implementation in codebase
   - Assess if implementation matches requirement intent
   - Flag divergences as WARNING

   **Scenario Coverage:**
   - For each scenario in specs (Given/When/Then), check code and test coverage
   - Flag uncovered scenarios as WARNING

5. **Verify Coherence**

   **Design Adherence:**
   - Extract key decisions from design.md
   - Verify implementation follows those decisions
   - Flag contradictions as WARNING

   **Code Pattern Consistency:**
   - Review new code for consistency with project patterns
   - Flag significant deviations as SUGGESTION

6. **Generate Verification Report**

## Output

```
## Verification Report: <change-name>

### Summary
| Dimension    | Status              |
|--------------|---------------------|
| Completeness | X/Y tasks, N reqs   |
| Correctness  | M/N reqs covered    |
| Coherence    | Followed / Issues   |

### Issues by Priority

**CRITICAL** (must fix before archive):
- [description with file:line and recommendation]

**WARNING** (should fix):
- [description with file:line and recommendation]

**SUGGESTION** (nice to fix):
- [description and recommendation]

### Final Assessment
[Ready for archive / Fix N critical issues first]
```

## Example

Input: `/opsx:verify add-user-notifications`

Output: Verification report showing 8/8 tasks complete, 5/6 requirements implemented, 1 WARNING about missing edge case scenario, SUGGESTION about inconsistent error handling pattern. Final assessment: "No critical issues. 1 warning to consider. Ready for archive."

## Guardrails

- Do NOT auto-select a change — always let the user choose if ambiguous
- Every issue must have a specific recommendation with file/line references
- Prefer SUGGESTION over WARNING, WARNING over CRITICAL when uncertain
- If only tasks.md exists, verify task completion only — skip spec/design checks
- Always note which checks were skipped and why
