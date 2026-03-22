# OpenSpec Workflow (Codex)

## Tool Positioning

- **OpenSpec** — Spec-alignment engine. Ensures human and AI agree on WHAT to build before writing code.
- **GSD is not available in Codex** — Tech Lead handles task decomposition and coordination directly.

## Scale-Based Strategy

| Scale | Effort | Strategy |
|-------|--------|----------|
| Micro | < 30 min | Direct code change |
| Small | 30 min ~ 2 hr | OpenSpec propose + manual implementation |
| Medium | 2 ~ 8 hr | OpenSpec propose + Tech Lead coordination + specialist agents |
| Large | > 1 day | Multi-session OpenSpec + phased coordination |

## Source of Truth

| Layer | Source | Location |
|-------|--------|----------|
| Requirements | OpenSpec specs | `openspec/specs/` + `openspec/changes/<name>/specs/` |
| Design | OpenSpec design | `openspec/changes/<name>/design.md` |
| Change History | OpenSpec archive | `openspec/changes/archive/` |

## Dual Verification

Medium+ features require:
1. Functional verification — all tasks complete, tests pass
2. Spec conformance — openspec-verify skill (completeness, correctness, coherence)

Both must pass before archiving.

## Violations

- Medium+ feature implemented without OpenSpec change
- Implementation started without specs confirmed by user
- Feature shipped without dual verification
