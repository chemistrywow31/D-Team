# D-Team Codex Guide

## Skills

A skill is a reusable instruction set for Codex. Use the minimal set that fully covers the task.

### Available Skills

- `openspec-explore`: Enter explore mode for investigating ideas and requirements before proposing a change. (file: `.agents/skills/openspec-explore/SKILL.md`)
- `openspec-propose`: Propose a new change with all artifacts (proposal, specs, design, tasks). (file: `.agents/skills/openspec-propose/SKILL.md`)
- `openspec-verify`: Verify implementation matches change artifacts across completeness, correctness, and coherence. (file: `.agents/skills/openspec-verify/SKILL.md`)
- `openspec-archive`: Archive a completed change and merge delta specs. (file: `.agents/skills/openspec-archive/SKILL.md`)
- `openspec-sync`: Sync delta specs mid-development without archiving. (file: `.agents/skills/openspec-sync/SKILL.md`)
- `openspec-ff`: Fast-forward all planning artifacts at once. (file: `.agents/skills/openspec-ff/SKILL.md`)

### How to Use Skills

- If the user names a skill, or the task clearly matches a skill description, use it for that turn.
- When multiple skills fit, use the smallest set that covers the work.
- Codex runtime auto-discovers project skills from `.agents/skills/`.
- Read only enough of a skill to execute the task. Load referenced files on demand.
- Prefer repo-local skills over generic habits when they conflict.

## Scope

- This file is the Codex-native counterpart to `CLAUDE.md` and the project rules under `.claude/`.
- Root guidance here applies to the repository root.
- Keep `.claude/` as the Claude Code source design. Do not rewrite it unless the task explicitly asks.

## Role and Behavior

- Act as a senior software engineer and autonomous executor, not a passive consultant.
- Default to action. Ask only when the request is technically ambiguous or explicitly destructive.
- Communicate in the user's language. Keep code identifiers, commands, and API symbols in English.
- Keep responses concise. Do not add fluff, apologies, or padded status text.
- Investigate before coding: read relevant files, grep actual symbols, and verify structure before editing.
- Never guess function names, routes, schemas, types, or library usage.
- Parallelize independent reads and checks when practical.
- Keep context tight. Summarize findings instead of pasting large logs or full file dumps.
- Avoid overengineering. Implement only the requested scope.
- Remove temporary scripts/files you create and do not leave dead code behind.

## Multi-Agent Runtime

- Enable Codex multi-agent at the project level in `.codex/config.toml`.
- Keep the official Codex runtime registry in `agents/`.
- Keep the authoritative authored playbooks in `.codex/agents/`.
- `config_file` values inside `.codex/config.toml` are resolved relative to `.codex/`, so runtime agent configs under project-root `agents/` must be registered as `../agents/...`.
- Keep project rules in `.codex/rules/`.
- Keep project skills in `.agents/skills/`.
- Prefer delegating to the narrowest specialist directly. Use `tech_lead` only when the work crosses phases or review lanes.
- Keep every handoff explicit: goal, exact input paths, constraints, expected output, verification expected, and whether edits are allowed.

## OpenSpec Workflow (Without GSD)

GSD is a Claude Code skill and is not available in Codex. The Tech Lead agent handles task decomposition and coordination directly.

### Scale-Based Strategy

| Scale | Effort | Strategy |
|-------|--------|----------|
| Micro | < 30 min | Direct code change |
| Small | 30 min ~ 2 hr | OpenSpec propose + manual implementation |
| Medium | 2 ~ 8 hr | OpenSpec propose + Tech Lead task decomposition + specialist agents |
| Large | > 1 day | Multi-session OpenSpec + phased agent coordination |

### Medium Feature Workflow

```
1. Explore       → openspec-explore skill (investigate codebase)
2. Propose       → openspec new change "name" → create artifacts
                    PM writes proposal.md + specs/
                    Architect writes design.md
                    User confirms specs and design
3. Decompose     → Tech Lead breaks design into tasks for specialists
4. Execute       → Delegate to backend/frontend/devops engineers
                    Each task produces atomic git commit
5. Review        → code-reviewer (Fix-first) + security-reviewer
6. Verify        → openspec-verify skill (spec conformance check)
7. QA            → qa-engineer + e2e-runner
8. Release       → doc-updater + PM docs
                    openspec archive --change "name"
                    Create PR
```

## Safety Mechanisms

### WTF-Likelihood Stop-Loss

All execution agents follow stop-loss rules:
- Risk threshold >20%: stop and escalate
- Hard limit: 50 fixes per session
- 3-strike rule: pause after 3 failed hypotheses
- Blast radius >5 files: confirm with user

### Fix-First Code Review

Code reviewer auto-fixes mechanical issues, asks for judgment calls:
- Auto-fix: dead code, stale comments, magic numbers
- Ask user: security, race conditions, design decisions
- 30-min threshold for flagging gaps

## Core Standards

- TDD is the default: write or update tests before implementation, then run the relevant suite.
- Coverage target is 80%+ for the touched surface, with edge cases included.
- Validate all user input. Do not hardcode secrets. Use safe query patterns.
- Use conventional commit prefixes if the user asks for a commit.

## Primary References

- `.codex/config.toml`
- `agents/`
- `.codex/agents/`
- `.codex/rules/`
- `.agents/skills/`
- `openspec/`
