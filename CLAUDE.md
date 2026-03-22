# D-Team (Dev Team)

## Team Objectives

D-Team is a full-stack development team that combines **OpenSpec** (spec-alignment engine) with **GSD** (execution engine) into a unified development pipeline. OpenSpec ensures human and AI agree on WHAT to build before writing code. GSD ensures execution quality through independent context windows, wave-based parallel execution, and session persistence.

All agents follow test-driven development, fix-first code review, and WTF-likelihood stop-loss mechanisms.

## Universal Standards

- **Language**: Communicate in the user's language. Technical terms remain in English.
- **Testing**: Minimum 80% test coverage. Write tests before implementation (TDD).
- **Security**: No hardcoded secrets. Validate all user input. Use parameterized queries.
- **Git**: Follow conventional commits (`feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`).
- **Code Review**: Every code change must pass code-reviewer before merge. Fix-first flow: auto-fix mechanical issues, ask for judgment calls.
- **Stop-Loss**: All execution agents follow WTF-likelihood rules — stop when fixes become too risky (>20% threshold).

## Tech Stack (Default)

This team is designed to be **tech-stack agnostic**. The default configuration targets full-stack web development. Adapt agent skills and rules for your specific stack. See `README.md` for customization guidance.

## Agent Organization

```
.claude/agents/
├── tech-lead.md              # Coordinator (orchestration only)
├── spec/                     # Spec & design agents
│   ├── product-manager.md    # Requirements, proposals, specs
│   └── architect.md          # System design, technical decisions
├── core/                     # Core development agents
│   ├── backend-engineer.md   # Backend implementation
│   ├── frontend-engineer.md  # Frontend implementation
│   └── devops-engineer.md    # Infrastructure, CI/CD
├── quality/                  # Review & QA agents
│   ├── code-reviewer.md      # Code review (Fix-first flow)
│   ├── security-reviewer.md  # Security audit
│   ├── qa-engineer.md        # Quality assurance & testing
│   └── process-reviewer.md   # Workflow retrospective
└── tools/                    # Utility agents
    ├── build-resolver.md     # Build/lint error fixes
    ├── e2e-runner.md         # E2E test execution
    ├── refactor-cleaner.md   # Dead code cleanup
    └── doc-updater.md        # Documentation generation
```

## Deployment Mode

### Subagent Mode (Default)

Agents are invoked via the Task tool within a single session. The Tech Lead coordinator manages all delegation through the OpenSpec + GSD integrated workflow. Use this mode for sequential feature development with clear handoffs.

### Agent Teams Mode (Experimental)

Agents run as independent Claude Code instances with shared task lists and direct messaging. Use this mode when multiple phases can execute in parallel (e.g., independent feature branches). Requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` to be enabled.

Communication topology in Agent Teams mode:
- **Coordinator-hub**: Tech Lead broadcasts phase transitions and assigns tasks
- **Peer review**: Quality agents can directly message development agents for review feedback
- **Escalation**: All agents escalate blocking issues to Tech Lead

## OpenSpec + GSD Integrated Workflow

### Tool Positioning

- **OpenSpec** — Spec-alignment engine. Ensures human and AI agree on WHAT to build before writing code.
- **GSD** — Execution engine. Uses independent context windows, wave-based parallel execution, and session persistence to maintain quality.

**One pipeline, not two**: OpenSpec defines WHAT, GSD handles HOW. Skip overlapping stages.

### Scale-Based Strategy

| Scale | Effort | Strategy |
|-------|--------|----------|
| Micro | < 30 min | Direct code or `/gsd:quick` |
| Small | 30 min ~ 2 hr | `/gsd:fast` or OpenSpec propose + manual |
| Medium | 2 ~ 8 hr | OpenSpec + GSD full integration (sweet spot) |
| Large | > 1 day | Multi-phase OpenSpec + GSD + session persistence |

### Medium Feature Workflow (Standard)

```
Phase 0: Explore      → /opsx:explore (optional, when tech is uncertain)
Phase 1: Propose      → /opsx:propose (PM + Architect: proposal → specs → design)
                        → User confirms specs and design
Phase 2: Plan         → /gsd:plan-phase N (reads OpenSpec specs + design)
Phase 3: Execute      → /gsd:execute-phase N (wave-based parallel, atomic commits)
                        → Code Review (Fix-first) + Security Review
Phase 4: Verify       → /gsd:verify-work N (functional completeness)
                        → /opsx:verify <change> (spec conformance)
Phase 5: QA           → QA Engineer + E2E Runner
Phase 6: Release      → doc-updater + PM docs
                        → /opsx:archive <change> (merge delta specs)
                        → /gsd:ship N (create PR)
```

### Source of Truth

| Layer | Source | Location |
|-------|--------|----------|
| Requirements (What) | OpenSpec | `openspec/specs/` + `openspec/changes/<name>/specs/` |
| Execution Plan (How) | GSD | `.planning/phases/XX-YY-PLAN.md` |
| Change History | OpenSpec | `openspec/changes/archive/` |
| Execution Progress | GSD | `.planning/STATE.md` + `ROADMAP.md` |

When GSD plan contradicts OpenSpec specs → specs win. Correct the plan, not the specs.

## Safety Mechanisms

### WTF-Likelihood Stop-Loss

All execution agents (core/ + tools/) follow the WTF-likelihood rule:
- **Risk threshold**: Stop if fixes become too risky (>20% WTF-likelihood)
- **Hard limit**: Maximum 50 fixes per session
- **3-strike rule**: After 3 failed hypotheses, pause and escalate to user
- **Blast radius check**: If a fix touches >5 files, confirm with user before proceeding
- **Scope lock**: After forming a hypothesis, restrict edits to the affected directory

See `rules/wtf-likelihood.md` for full details.

### Fix-First Code Review

Code reviewer uses fix-first flow:
- **Auto-fix**: Mechanical issues (dead code, stale comments, magic numbers) are fixed automatically with `[AUTO-FIXED]` attribution
- **Ask user**: Judgment calls (security, race conditions, design decisions) are surfaced for human decision
- **30-min threshold**: Only flag gaps if the 100% solution costs less than 30 minutes of agent time

See `rules/fix-first-review.md` for full details.

## Worklog & Context Management

### Worklog Structure

All work is documented in `.worklog/yyyymm/task-name/phase-n-label/` with three core files per phase:
- `references.md` — Sources consulted
- `findings.md` — Key discoveries and analysis
- `decisions.md` — Decisions with rationale, alternatives, and evidence chain

Evidence chain: references → findings → decisions. Every decision must trace back through findings to references.

### Context Management

- **Coordinator dispatch**: Every Task dispatch must include worklog path and upstream reference paths
- **Agent return format**: Agents return structured summaries; full detail goes to the worklog
- **Task isolation**: Each GSD task runs in an independent context window (no cross-contamination)
- **Phase-end archival**: Coordinator verifies worklog completeness before phase transitions
- **Context recovery**: Read latest phase worklog to restore context after interruption

### Summary-Based Reporting

Agents must report results as concise summaries containing:
- What was done (one to three sentences)
- Files created or modified (list of paths)
- Decisions made and rationale (bullet points)
- Open issues or blockers (if any)

Do not dump full file contents, complete logs, or raw command output into reports.
