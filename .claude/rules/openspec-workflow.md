---
name: OpenSpec GSD Integrated Workflow
description: Unified development workflow combining OpenSpec spec-alignment with GSD execution management
---

# OpenSpec + GSD Integrated Workflow

## Applicability

- Applies to: `tech-lead`, `product-manager`, `architect`, `backend-engineer`, `frontend-engineer`, `code-reviewer`, `spec-test-auditor`
- Scope: All new feature development and requirement-driven changes

## Rule Content

### Tool Positioning

- **OpenSpec** — Spec-alignment engine. Solves "AI builds the wrong thing". Ensures human and AI agree on WHAT to build before writing code.
- **GSD** — Execution engine. Solves "AI quality degrades mid-session". Uses independent context windows, wave-based parallel execution, and session persistence to maintain quality.

**One pipeline, not two**: OpenSpec defines WHAT, GSD handles HOW. Skip overlapping stages — use OpenSpec specs instead of GSD discuss, use GSD executor instead of OpenSpec apply.

### Source of Truth Principle

| Layer | Source of Truth | Location |
|-------|----------------|----------|
| Requirements (What) | OpenSpec specs | `openspec/specs/` and `openspec/changes/<name>/specs/` |
| Execution Plan (How) | GSD plan | `.planning/phases/XX-YY-PLAN.md` |
| Change History | OpenSpec archive | `openspec/changes/archive/` |
| Execution Progress | GSD state | `.planning/STATE.md` + `ROADMAP.md` |

**Conflict resolution**: When GSD plan content contradicts OpenSpec specs, specs win. Return to OpenSpec to confirm whether requirements need adjustment. If requirements are correct, modify the GSD plan.

### Scale-Based Strategy Selection

Select the workflow strategy based on estimated effort. Do not run full pipeline for trivial changes.

#### Micro (< 30 min) — Bug fix, typo, config

```
Direct code changes
  or
/gsd:quick "fix description"
```

No OpenSpec required. Writing specs takes longer than the fix itself.

#### Small (30 min ~ 2 hr) — Single endpoint, form field, utility

**Option A** — GSD fast mode (clear solution in mind):
```
/gsd:fast "feature description"
```

**Option B** — OpenSpec alignment + manual implementation (behavior details need clarification):
```
/opsx:propose "feature description"
→ Confirm proposal + specs
→ Implement manually based on specs
/opsx:archive <change-name>
```

#### Medium (2 ~ 8 hr) — Full module, subsystem refactor, third-party integration

This is the integration sweet spot. OpenSpec ensures correct direction, GSD ensures efficient execution.

```
/opsx:explore "technical investigation"       ← Optional, when tech is uncertain
/opsx:propose "feature description"           ← Produce specs + design
→ Confirm specs and design with user
/gsd:plan-phase N                             ← GSD plans based on specs
/gsd:execute-phase N                          ← Wave-based parallel execution
/gsd:verify-work N                            ← Functional completeness check
/opsx:verify <change-name>                    ← Spec conformance check
/opsx:archive <change-name>                   ← Archive delta specs
/gsd:ship N                                   ← Create PR
```

#### Large (> 1 day) — Architecture migration, cross-system refactor

```
Phase 0: Research
  /opsx:explore "feasibility analysis"
  /gsd:map-codebase                           ← 4-way parallel codebase analysis

Phase 1: Spec Definition
  /opsx:propose "feature description"
  → Confirm proposal, specs, design iteratively

Phase 2: Multi-phase Planning
  Split OpenSpec design into multiple GSD phases
  /gsd:plan-phase 1                           ← Infrastructure
  /gsd:plan-phase 2                           ← Core logic
  /gsd:plan-phase 3                           ← Integration + testing

Phase 3: Phased Execution (repeat per phase)
  /gsd:execute-phase N
  /gsd:verify-work N
  /opsx:verify <change>                       ← Spec conformance per phase

Phase 4: Wrap-up
  /opsx:archive <change>
  /gsd:ship N
```

Use `/gsd:pause-work` + `/gsd:resume-work` for cross-session persistence.

### Artifact Ownership During a Change

| Artifact | Owner | Purpose |
|----------|-------|---------|
| `proposal.md` | PM | What & why — business justification |
| `specs/<capability>/spec.md` | PM | Detailed requirements per capability |
| `design.md` | Architect | Technical design — how to implement |
| `tasks.md` | Tech Lead | Implementation steps (reference when GSD plans execution) |
| `.planning/phases/XX-YY-PLAN.md` | GSD Planner | Executable task plan (medium+ features) |

### Default Single + Conditional Dual Verification

Phase 4 verification uses a layered strategy to avoid redundant 80%+ overlap between `/gsd:verify-work` and `/opsx:verify` when automated verify already references OpenSpec Gherkin scenarios.

**Default layer** (always runs):
- `/gsd:verify-work N` — Checks functional completeness: all task plan `<done>` criteria met, automated tests pass. Reads the `<verify><automated>` block in PLAN.md, which MUST reference OpenSpec Gherkin scenarios for spec-driven features.

**Conditional layer** (runs ONLY when a contract-layer change is detected):
- `/opsx:verify <change>` — Checks spec conformance from three dimensions:
  - **Completeness** — Every MUST/SHOULD requirement in specs has corresponding implementation
  - **Correctness** — Implementation behavior matches Given/When/Then scenarios
  - **Consistency** — No contradictions between modules

When contract-layer detection conditions below are NOT met, skip `/opsx:verify` — the GSD verify already covers spec conformance via its automated block. When they ARE met, both layers must pass before archive/ship.

### Contract-Layer Change Detection

Trigger `/opsx:verify` as the conditional dual-verification layer when ANY of the following applies:

1. **API schema changes** — Request or response shape modifications, endpoint additions or removals, HTTP status code or header contract changes, GraphQL schema mutations, RPC signature changes.
2. **Data model changes** — Schema migrations, new or removed fields, type changes on existing fields, index or constraint changes that alter observable data contracts, enum value additions or removals.
3. **Cross-change spec interactions** — The current change depends on or modifies another change's spec (detected by inspecting `openspec/changes/<name>/specs/` for references to other change names, or by multi-change delta overlap in a single merge cycle).

Detection procedure the Tech Lead runs before Phase 4:
- Inspect `git diff` for files under `openspec/specs/`, API route files, schema or migration directories.
- Inspect `openspec/changes/<name>/specs/` for cross-references to other changes.
- If any condition above is met → run `/opsx:verify` in addition to `/gsd:verify-work`.
- If none are met → run `/gsd:verify-work` only and record the skip reason in the Phase 4 worklog (`decisions.md`).

### How Agents Interact with OpenSpec + GSD

| Agent | Reads | Writes |
|-------|-------|--------|
| Tech Lead | All artifacts, `openspec status`, `.planning/STATE.md` | tasks.md, GSD orchestration commands |
| PM | Existing specs, user docs | proposal.md, specs/*.md |
| Architect | proposal.md, specs/*.md | design.md |
| Backend Engineer | design.md, GSD plan | Code (marks tasks complete) |
| Frontend Engineer | design.md, GSD plan | Code (marks tasks complete) |
| Code Reviewer | All artifacts (for context) | Review comments only |
| Spec Test Auditor | specs/*.md, GSD plan, ROADMAP.md | Behavioral test cases, coverage gap report |

## Violation Determination

- New feature (medium+ scale) implemented without OpenSpec change → Violation
- Medium+ feature uses `/opsx:apply` instead of GSD plan+execute → Violation
- Implementation started without specs confirmed by user → Violation
- Medium+ feature with a contract-layer change shipped without running conditional `/opsx:verify` → Violation
- Tech Lead skipping `/opsx:verify` on a contract-layer change without documenting the detection decision in the Phase 4 worklog → Violation
- GSD plan contradicts OpenSpec specs and plan was not corrected → Violation
- Tasks marked complete without corresponding code changes → Violation

## Exceptions

- Trivial bug fixes (one-line changes, typos) do not require OpenSpec or GSD
- Hotfixes for production incidents may bypass the full workflow but must be retroactively documented
- Refactoring that does not change behavior may use `/gsd:fast` without OpenSpec
- Small features (< 2 hr) may use either OpenSpec-only or GSD-only workflow based on team judgment
