# D-Team (Dev Team)

> **[English](#english)** | **[繁體中文](#繁體中文)**

---

<a id="english"></a>

## English

A full-stack development team powered by **OpenSpec + GSD** integrated workflow, with **WTF-likelihood stop-loss** and **Fix-first code review** mechanisms.

Supports both **Claude Code** and **Codex** runtimes.

### Table of Contents

- [What This Team Does](#what-this-team-does)
- [Current Tech Stack](#current-tech-stack-default)
- [Platform Setup](#platform-setup)
  - [Claude Code](#option-a-claude-code)
  - [Codex](#option-b-codex)
- [Getting Started](#getting-started)
  - [Greenfield Project](#greenfield-project-starting-from-scratch)
  - [Brownfield Project](#brownfield-project-existing-codebase)
- [Development Workflow](#development-workflow)
- [Team Agents](#team-agents-14)
- [Safety Mechanisms](#safety-mechanisms)
- [Adding Tech-Stack Skills](#adding-tech-stack-skills)
- [Directory Structure](#directory-structure)

### What This Team Does

D-Team provides a complete development lifecycle from requirements to deployment:

1. **Spec Alignment** (OpenSpec) — Ensure human and AI agree on what to build before writing code
2. **Execution Management** (GSD) — Wave-based parallel execution with independent context windows
3. **Quality Assurance** — Fix-first code review, security audit, TDD enforcement
4. **Safety Mechanisms** — WTF-likelihood stop-loss prevents runaway auto-fixing

### Current Tech Stack (Default)

This team ships with a **tech-stack agnostic** configuration. Agent skills reference generic patterns that work across stacks. For production use, customize the skills to match your project.

| Layer | Default Assumption | Customization Point |
|-------|-------------------|---------------------|
| Backend | Any server-side language (Go, Node.js, Python, etc.) | `.claude/agents/core/backend-engineer.md` — Add language-specific skills |
| Frontend | Any frontend framework (React, Vue, Svelte, etc.) | `.claude/agents/core/frontend-engineer.md` — Add framework-specific skills |
| Database | Any database (PostgreSQL, MongoDB, MySQL, etc.) | Add `security-reviewer` DB-specific checks |
| Infrastructure | Docker + CI/CD | `.claude/agents/core/devops-engineer.md` |
| Testing | Language-native test frameworks | `.claude/rules/tdd-enforcement.md` — Update commands |

---

### Platform Setup

#### Option A: Claude Code

##### Prerequisites

| Dependency | Install | Purpose |
|-----------|---------|---------|
| [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) | `npm install -g @anthropic-ai/claude-code` | AI coding agent runtime |
| [OpenSpec CLI](https://github.com/openspec-dev/openspec) | `npm install -g openspec` | Spec-alignment engine (proposal → specs → design → tasks) |
| [GSD skill](https://github.com/wmayner/gsd) | Install via Claude Code: `/skill-install gsd` | Execution engine (plan → execute → verify → ship) |

##### Installation

```bash
# 1. Clone D-Team into your project
git clone https://github.com/anthropics/D-Team.git d-team-source
cp -r d-team-source/.claude/ /path/to/your-project/.claude/
cp d-team-source/CLAUDE.md /path/to/your-project/CLAUDE.md
rm -rf d-team-source

# 2. Initialize OpenSpec in your project
cd /path/to/your-project
openspec init
# Edit openspec/config.yaml — set project name, description, and constraints

# 3. Verify setup
claude   # Start Claude Code
# Type: /opsx:explore "test"
# Should enter explore mode without errors
```

##### GSD Usage Notes

GSD provides its own internal agents (`gsd-planner`, `gsd-executor`, `gsd-verifier`, etc.) that handle execution orchestration. D-Team's agents handle the **domain work** (writing code, reviewing, testing). The separation:

| Layer | Handled By | Examples |
|-------|-----------|---------|
| **Execution orchestration** | GSD internal agents | Task planning, wave parallelization, state tracking, atomic commits |
| **Domain work** | D-Team agents | Writing code, code review, security audit, testing, documentation |

**Important**: GSD's `/gsd:discuss-phase` is **skipped** in the D-Team workflow because OpenSpec specs already cover the "what to build" discussion. Use this flow instead:

```
/opsx:propose → (confirms specs) → /gsd:plan-phase → /gsd:execute-phase
```

If GSD prompts you to run `discuss-phase`, you can skip it with `--auto` or go directly to `plan-phase`.

##### Claude Code Workflow Commands

```bash
# OpenSpec commands (spec alignment)
/opsx:explore "topic"              # Think through a problem
/opsx:propose "feature-name"       # Create proposal + specs + design
/opsx:verify "change-name"         # Verify implementation matches specs
/opsx:archive "change-name"        # Archive completed change
/opsx:sync "change-name"           # Sync delta specs mid-development
/opsx:ff "name" "description"      # Fast-forward all artifacts at once

# GSD commands (execution management)
/gsd:quick "description"           # Micro fix (< 30 min)
/gsd:fast "description"            # Small feature (30 min ~ 2 hr)
/gsd:plan-phase N                  # Plan a phase from specs
/gsd:execute-phase N               # Wave-based parallel execution
/gsd:verify-work N                 # Functional verification
/gsd:ship N                        # Create PR
/gsd:pause-work                    # Save session state
/gsd:resume-work                   # Restore session state
/gsd:map-codebase                  # 4-way codebase analysis
```

---

#### Option B: Codex

##### Prerequisites

| Dependency | Install | Purpose |
|-----------|---------|---------|
| [Codex CLI](https://github.com/openai/codex) | `npm install -g @openai/codex` | AI coding agent runtime (OpenAI) |
| [OpenSpec CLI](https://github.com/openspec-dev/openspec) | `npm install -g openspec` | Spec-alignment engine |

> **Note**: GSD is a Claude Code skill and is **not available in Codex**. The Codex workflow uses OpenSpec for spec alignment and manual/agent-driven execution instead of GSD's wave-based parallelism. The Tech Lead agent handles task decomposition and coordination directly.

##### Installation

```bash
# 1. Clone D-Team and copy Codex files to your project
git clone https://github.com/anthropics/D-Team.git d-team-source
cp -r d-team-source/agents/ /path/to/your-project/agents/
cp -r d-team-source/.codex/ /path/to/your-project/.codex/
cp d-team-source/AGENTS.md /path/to/your-project/AGENTS.md
cp -r d-team-source/.claude/skills/ /path/to/your-project/.agents/skills/
# Note: Codex discovers skills from .agents/skills/ at runtime
rm -rf d-team-source

# 2. Initialize OpenSpec
cd /path/to/your-project
openspec init

# 3. Verify setup
codex   # Start Codex
# Ask: "List available agents"
# Should show all 14 agents
```

##### Codex vs Claude Code Differences

| Feature | Claude Code | Codex |
|---------|------------|-------|
| Agent dispatch | Task tool (subagent mode) | Multi-agent threads (`.codex/config.toml`) |
| GSD execution | `/gsd:*` commands with wave parallelism | Not available — Tech Lead coordinates manually |
| OpenSpec skills | `/opsx:*` slash commands | Read SKILL.md and follow steps manually |
| Session persistence | `/gsd:pause-work` / `resume-work` | Manual handoff notes |
| Model selection | `opus` / `sonnet` / `haiku` | Configure `model` in `.toml` files |
| Review agents | `read-only` via Task tool | `sandbox_mode = "read-only"` in `.toml` |

##### Codex Workflow (Without GSD)

```
1. Explore      → Read openspec-explore skill, investigate codebase
2. Propose      → openspec new change "name" → create artifacts manually
3. Plan         → Tech Lead decomposes tasks from design.md
4. Execute      → Delegate to backend/frontend engineers via agents
5. Review       → code-reviewer + security-reviewer
6. Verify       → Read openspec-verify skill, run verification
7. Ship         → Create PR, archive change
```

##### Codex Model Configuration

Edit `agents/*.toml` to set models. Default configuration:

| Agent Type | Default Model | Reasoning Effort | Sandbox |
|-----------|---------------|-----------------|---------|
| Coordinator | `o3` | `high` | `workspace-write` |
| Spec agents | `o3` | `high` | `workspace-write` |
| Core engineers | `o3` | `high` | `workspace-write` |
| Quality reviewers | `o3` | `high` | `read-only` |
| Tools agents | `o3` | `medium` | `workspace-write` |

---

### Getting Started

#### Greenfield Project (Starting from Scratch)

```
Day 0 — Bootstrap
│
├─ 1. Create project repo, install dependencies
├─ 2. Install D-Team (see Platform Setup above)
├─ 3. Initialize OpenSpec
│     $ openspec init
│     Edit openspec/config.yaml:
│       - project name and description
│       - tech stack constraints (language, framework, DB)
│       - team conventions (coding style, commit format)
│
├─ 4. Create tech-stack skills (see "Adding Tech-Stack Skills")
│     .claude/skills/your-language-patterns/SKILL.md
│     .claude/skills/your-framework-patterns/SKILL.md
│
├─ 5. Wire skills to agents
│     Update backend-engineer.md, frontend-engineer.md, code-reviewer.md
│
└─ 6. First feature
      $ claude
      /opsx:explore "what this project needs to do"
      /opsx:propose "initial-setup"
      → Architect produces design.md with project structure, API contracts, data model
      → User confirms
      /gsd:plan-phase 1
      /gsd:execute-phase 1
```

**Greenfield tips:**
- Spend time on `openspec/config.yaml` — this is the project's DNA that all agents read for context
- The first `/opsx:propose` should cover project scaffolding: directory structure, build tooling, CI pipeline, base configs
- Let the Architect define foundational patterns early — all subsequent code inherits from these decisions
- Run `/opsx:archive` after the initial setup to lock in the base specs as the project's long-term reference

#### Brownfield Project (Existing Codebase)

```
Day 0 — Integration
│
├─ 1. Install D-Team into your existing project (see Platform Setup)
│
├─ 2. Initialize OpenSpec with existing context
│     $ openspec init
│     Edit openspec/config.yaml:
│       - describe existing architecture, tech stack, conventions
│       - note any areas of technical debt or constraints
│       - reference key architecture docs if they exist
│
├─ 3. Map the codebase
│     $ claude
│     /gsd:map-codebase
│     → Produces 4-way analysis: tech stack, architecture, quality, concerns
│     → Read the output to understand what D-Team is working with
│
├─ 4. Create tech-stack skills from your existing patterns
│     Read your codebase, extract the patterns your team already follows
│     Write them as skill files so agents match your conventions
│
├─ 5. Baseline your specs (optional but recommended)
│     /opsx:explore "current system architecture"
│     → Document existing features as specs in openspec/specs/
│     → This gives agents context about what already exists
│
└─ 6. First change
      /opsx:propose "your-first-change"
      → Agents read existing code + specs to understand boundaries
      → Architect's design.md respects existing patterns
      → Proceed with normal workflow
```

**Brownfield tips:**
- `/gsd:map-codebase` is essential — without it, agents may create duplicates or violate existing patterns
- Create skills that capture your project's existing conventions, not idealized ones
- Start with a small, well-scoped change to validate the workflow before tackling large features
- If your project has architecture docs, reference them in `openspec/config.yaml` so agents inherit that context
- Existing tests are your safety net — make sure `tdd-enforcement.md` has the correct test commands before agents start writing code

---

### Development Workflow

#### Full Lifecycle (Medium Feature, Claude Code)

```
┌─────────────────────────────────────────────────────────────────┐
│                        DEVELOPMENT LIFECYCLE                     │
└─────────────────────────────────────────────────────────────────┘

Phase 0: EXPLORE (optional)
┌──────────────────────────────────────────────┐
│  /opsx:explore "problem description"          │
│                                               │
│  → Investigate codebase                       │
│  → ASCII diagrams, trade-off analysis         │
│  → Surface risks and unknowns                 │
│  → No code written — thinking only            │
│                                               │
│  Output: Shared understanding of the problem  │
└──────────────────┬───────────────────────────┘
                   ▼
Phase 1: PROPOSE (spec alignment)
┌──────────────────────────────────────────────┐
│  /opsx:propose "feature-name"                 │
│                                               │
│  PM writes:                                   │
│    ├─ proposal.md     (what & why)            │
│    └─ specs/          (detailed requirements) │
│  Architect writes:                            │
│    └─ design.md       (how — contracts,       │
│                        data model, diagrams)  │
│                                               │
│  ⚠ USER CONFIRMS specs + design before next   │
│                                               │
│  Output: openspec/changes/<name>/             │
└──────────────────┬───────────────────────────┘
                   ▼
Phase 2: PLAN (execution planning)
┌──────────────────────────────────────────────┐
│  /gsd:plan-phase N                            │
│                                               │
│  GSD reads OpenSpec specs + design            │
│  Produces task plan with:                     │
│    ├─ <files> per task                        │
│    ├─ <verify> criteria                       │
│    └─ <done> conditions                       │
│  Plan quality validated (up to 3 iterations)  │
│                                               │
│  Output: .planning/phases/XX-YY-PLAN.md       │
└──────────────────┬───────────────────────────┘
                   ▼
Phase 3: EXECUTE (implementation)
┌──────────────────────────────────────────────┐
│  /gsd:execute-phase N                         │
│                                               │
│  Wave-based parallel execution:               │
│    Wave 1: [Task A] [Task B] [Task C]  ←parallel│
│    Wave 2: [Task D] [Task E]           ←parallel│
│    Wave 3: [Task F]                            │
│                                               │
│  Each task:                                   │
│    ├─ Independent 200K context window         │
│    ├─ TDD: write test → implement → refactor  │
│    ├─ WTF-likelihood check before each fix    │
│    └─ Atomic git commit on completion         │
│                                               │
│  Then: code-reviewer (Fix-first flow)         │
│        security-reviewer (if auth/API changes)│
│                                               │
│  Output: Code + tests + review verdicts       │
└──────────────────┬───────────────────────────┘
                   ▼
Phase 4: VERIFY (dual verification)
┌──────────────────────────────────────────────┐
│  /gsd:verify-work N                           │
│    → Functional: all <done> criteria met?     │
│    → Tests pass?                              │
│                                               │
│  /opsx:verify <change-name>                   │
│    → Completeness: all requirements covered?  │
│    → Correctness: behavior matches scenarios? │
│    → Coherence: design decisions followed?    │
│                                               │
│  ⚠ BOTH must pass before proceeding           │
│                                               │
│  Output: Verification reports                 │
└──────────────────┬───────────────────────────┘
                   ▼
Phase 5: QA (testing)
┌──────────────────────────────────────────────┐
│  qa-engineer                                  │
│    → Write test cases from OpenSpec scenarios │
│    → Verify edge cases and error paths        │
│                                               │
│  e2e-runner                                   │
│    → Execute critical user flow tests         │
│    → Diagnose failures (test bug vs app bug)  │
│                                               │
│  If REJECTED → back to Phase 3 with defects   │
│                                               │
│  Output: QA report (APPROVED / REJECTED)      │
└──────────────────┬───────────────────────────┘
                   ▼
Phase 6: RELEASE
┌──────────────────────────────────────────────┐
│  doc-updater     → Update architecture docs   │
│  product-manager → Update user-facing docs    │
│  process-reviewer → Retrospective report      │
│                                               │
│  /opsx:archive <change-name>                  │
│    → Move to archive/, merge delta specs      │
│                                               │
│  /gsd:ship N                                  │
│    → Create PR with auto-populated summary    │
│                                               │
│  Output: PR ready for merge                   │
└──────────────────────────────────────────────┘
```

#### Lifecycle Variants

**Micro fix (< 30 min):**
```
/gsd:quick "fix the bug"  →  done
```

**Small feature (30 min ~ 2 hr):**
```
/gsd:fast "add endpoint"  →  code-reviewer  →  done
  or
/opsx:propose "feature"  →  implement manually  →  /opsx:archive
```

**Large feature (> 1 day):**
```
/opsx:explore  →  /opsx:propose  →  /gsd:plan-phase 1..N
  →  for each phase:
       /gsd:execute-phase  →  /gsd:verify-work  →  /opsx:verify
  →  /opsx:archive  →  /gsd:ship
  →  use /gsd:pause-work + /gsd:resume-work across sessions
```

#### What Happens When Things Go Wrong

| Situation | What Triggers | What Happens |
|-----------|--------------|-------------|
| Agent fix is too risky | WTF-likelihood >20% | Agent stops, presents risk assessment, asks user |
| Agent keeps failing | 3 failed hypotheses | Agent stops, presents what it tried, asks for direction |
| Fix is too broad | Blast radius >5 files | Agent stops, asks user to confirm scope |
| Code review finds critical issue | CRITICAL finding | Review verdict = BLOCK, must fix before merge |
| Spec verification fails | Missing requirements | Back to Phase 3 to implement missing pieces |
| QA rejects | CRITICAL test failures | Back to Phase 3 with defect report |
| Build breaks | Compilation/type error | `build-resolver` called automatically |
| Session interrupted mid-work | User needs to stop | `/gsd:pause-work` saves full state for next session |
| GSD plan contradicts specs | Conflict detected | Specs win — plan is corrected, not the specs |

---

### Recommended Usage by Feature Size

| Size | Effort | Claude Code Flow | Codex Flow |
|------|--------|-----------------|------------|
| **Micro** | < 30 min | `/gsd:quick "fix"` | Direct code change |
| **Small** | 30 min ~ 2 hr | `/gsd:fast "feature"` | OpenSpec propose + manual impl |
| **Medium** | 2 ~ 8 hr | OpenSpec + GSD full pipeline | OpenSpec + Tech Lead coordination |
| **Large** | > 1 day | Multi-phase OpenSpec + GSD | Multi-session with handoff notes |

### Team Agents (14)

| Group | Agent | Claude Model | Codex Sandbox | Purpose |
|-------|-------|-------------|---------------|---------|
| Coordinator | `tech-lead` | opus | workspace-write | Orchestration, workflow management |
| Spec | `product-manager` | sonnet | workspace-write | Requirements, proposals, specs |
| Spec | `architect` | opus | workspace-write | System design, tech decisions |
| Core | `backend-engineer` | sonnet | workspace-write | Backend implementation |
| Core | `frontend-engineer` | sonnet | workspace-write | Frontend implementation |
| Core | `devops-engineer` | sonnet | workspace-write | Infrastructure, CI/CD |
| Quality | `code-reviewer` | opus | read-only | Code review (Fix-first flow) |
| Quality | `security-reviewer` | sonnet | read-only | Security audit |
| Quality | `qa-engineer` | sonnet | read-only | Test cases, UAT |
| Quality | `process-reviewer` | sonnet | read-only | Workflow retrospective |
| Tools | `build-resolver` | haiku | workspace-write | Build/lint error fixes |
| Tools | `e2e-runner` | haiku | workspace-write | E2E test execution |
| Tools | `refactor-cleaner` | haiku | workspace-write | Dead code cleanup |
| Tools | `doc-updater` | haiku | workspace-write | Documentation generation |

### Safety Mechanisms

#### WTF-Likelihood Stop-Loss (`rules/wtf-likelihood.md`)

Prevents execution agents from making things worse:
- **>20% risk threshold** — Stop if fixes become too risky
- **50 fixes hard limit** — Maximum fixes per session
- **3-strike rule** — Pause after 3 failed hypotheses
- **5-file blast radius** — Confirm with user if fix spans >5 files

#### Fix-First Review (`rules/fix-first-review.md`)

Code reviewer separates mechanical fixes from judgment calls:
- **Auto-fix** (no user approval needed): Dead code, stale comments, magic numbers, N+1 queries
- **Ask user** (requires approval): Security decisions, race conditions, design choices
- **30-min threshold**: Only flag gaps if the fix costs < 30 min of agent time

### Adding Tech-Stack Skills

D-Team ships with no language or framework opinions. You bring the stack knowledge by creating **skill files** — reusable instruction sets that agents load before doing their work.

#### Skill File Format

Every skill lives in its own folder with a `SKILL.md` file:

```
.claude/skills/{skill-name}/SKILL.md       ← Claude Code
.agents/skills/{skill-name}/SKILL.md       ← Codex (auto-discovered)
```

A skill file follows this structure:

```markdown
---
name: {Skill Name}
description: {One sentence — when to use this skill}
---

# {Skill Name}

## When to Use
{Conditions under which an agent should load this skill}

## Patterns
### {Pattern Category 1}
{Concrete code patterns, conventions, do/don't examples}

## Commands
{Build, test, lint, format commands relevant to this stack}

## Anti-Patterns
{Common mistakes to avoid, with reasoning}
```

#### What to Put in a Skill

| Content Type | Good Fit for Skill | Bad Fit (Use Rules Instead) |
|-------------|-------------------|---------------------------|
| Language idioms and conventions | Yes | — |
| Framework-specific patterns | Yes | — |
| Build/test/lint commands | Yes | — |
| Library usage patterns | Yes | — |
| Anti-patterns with reasoning | Yes | — |
| Process constraints (who does what) | — | Use `rules/` |
| Quality gates (coverage thresholds) | — | Use `rules/` |
| Workflow sequences | — | Use `rules/` |

#### Step-by-Step: Adding a New Stack

**1. Create skill files** — e.g., `.claude/skills/golang-patterns/SKILL.md` and `.claude/skills/postgres-patterns/SKILL.md`

**2. Wire skills to agents** — Add `## Required Skills` section to `backend-engineer.md`, `code-reviewer.md`, etc.

**3. Update TDD commands** — In `rules/tdd-enforcement.md`, set your stack's test/coverage commands

**4. (Optional) Add domain-specific reviewers** — e.g., `database-reviewer.md`

**5. Mirror for Codex** — Copy skills to `.agents/skills/` and update `AGENTS.md`

#### Quick Reference: Which Agents Need Which Skills

| Skill Category | Wire to These Agents |
|---------------|---------------------|
| Backend language (Go, Python, Rust, etc.) | `backend-engineer`, `code-reviewer` |
| Frontend framework (React, Vue, Svelte, etc.) | `frontend-engineer`, `code-reviewer` |
| Database (PostgreSQL, MongoDB, etc.) | `backend-engineer`, `database-reviewer` (if created) |
| Infrastructure (Docker, K8s, Terraform, etc.) | `devops-engineer` |
| Security (OWASP, auth patterns) | `security-reviewer` |
| Testing framework (Jest, pytest, etc.) | `qa-engineer`, `e2e-runner` |

#### Examples by Stack

| Stack | Skills to Create | Agents to Update |
|-------|-----------------|------------------|
| **Go + PostgreSQL** | `golang-patterns`, `postgres-patterns` | backend-engineer, code-reviewer, +database-reviewer |
| **TypeScript + MongoDB** | `typescript-patterns`, `mongodb-patterns` | backend-engineer, code-reviewer, +database-reviewer |
| **Python + FastAPI** | `python-patterns`, `fastapi-patterns` | backend-engineer, code-reviewer |
| **React / Next.js** | `react-patterns`, `nextjs-patterns` | frontend-engineer, code-reviewer |
| **Rust + SQLite** | `rust-patterns`, `sqlite-patterns` | backend-engineer, code-reviewer |
| **Vue + Supabase** | `vue-patterns`, `supabase-patterns` | frontend-engineer, code-reviewer |

### Directory Structure

#### Claude Code Layout

```
your-project/
├── CLAUDE.md                          ← Team-wide instructions
├── openspec/                          ← OpenSpec artifacts
│   ├── config.yaml
│   ├── specs/
│   └── changes/
├── .planning/                         ← GSD execution state
├── .worklog/                          ← Phase-level documentation
└── .claude/
    ├── agents/
    ├── skills/
    └── rules/
```

#### Codex Layout

```
your-project/
├── AGENTS.md                          ← Codex team-wide instructions
├── openspec/                          ← OpenSpec artifacts (shared)
├── .worklog/                          ← Phase-level documentation (shared)
├── agents/                            ← Codex runtime configs (.toml)
│   ├── tech-lead.toml
│   ├── core/
│   ├── quality/
│   └── tools/
├── .codex/
│   ├── config.toml                    ← Multi-agent config
│   ├── agents/                        ← Agent playbooks (.md)
│   └── rules/                         ← Codex rules
└── .agents/
    └── skills/                        ← Shared skills (auto-discovered)
```

#### Dual-Platform Layout (Both Claude Code + Codex)

```
your-project/
├── CLAUDE.md                          ← Claude Code entry point
├── AGENTS.md                          ← Codex entry point
├── openspec/                          ← Shared
├── .worklog/                          ← Shared
├── .claude/                           ← Claude Code configs
├── .codex/                            ← Codex configs
├── agents/                            ← Codex runtime (.toml)
└── .agents/skills/                    ← Codex skill discovery
```

---

---

<a id="繁體中文"></a>

## 繁體中文

結合 **OpenSpec + GSD** 整合式開發流程的全端開發團隊，內建 **WTF-likelihood 停損機制**與 **Fix-first 程式碼審查**。

同時支援 **Claude Code** 與 **Codex** 兩種執行環境。

### 目錄

- [這個團隊做什麼](#這個團隊做什麼)
- [預設技術棧](#預設技術棧)
- [環境建置](#環境建置)
  - [Claude Code](#方案-aclaude-code)
  - [Codex](#方案-bcodex)
- [專案起手式](#專案起手式)
  - [全新專案](#全新專案greenfield)
  - [既有專案](#既有專案brownfield)
- [開發流程](#開發流程)
- [團隊 Agents](#團隊-agents14-個)
- [安全機制](#安全機制)
- [建立技術棧 Skill](#建立技術棧-skill)
- [目錄結構](#目錄結構)

### 這個團隊做什麼

D-Team 涵蓋從需求定義到部署的完整開發生命週期：

1. **規格對齊**（OpenSpec）— 在寫程式之前，先確保人與 AI 對「要做什麼」達成共識
2. **執行管理**（GSD）— 以波次平行執行、獨立 context window、session 持久化維持品質
3. **品質保障** — Fix-first 程式碼審查、安全稽核、TDD 強制
4. **安全機制** — WTF-likelihood 停損防止 AI 越修越糟

### 預設技術棧

D-Team 出廠時**不綁定任何技術棧**，所有 agent 的 skill 採用通用模式。實際使用時，依你的專案技術棧建立對應的 skill 檔案即可。

| 層級 | 預設假設 | 客製化位置 |
|------|---------|-----------|
| 後端 | 任意伺服器端語言 | `.claude/agents/core/backend-engineer.md` — 加入語言專屬 skill |
| 前端 | 任意前端框架 | `.claude/agents/core/frontend-engineer.md` — 加入框架專屬 skill |
| 資料庫 | 任意資料庫 | 建立 `database-reviewer` agent |
| 基礎設施 | Docker + CI/CD | `.claude/agents/core/devops-engineer.md` |
| 測試 | 語言原生測試框架 | `.claude/rules/tdd-enforcement.md` — 更新指令 |

---

### 環境建置

#### 方案 A：Claude Code

##### 必要安裝

| 套件 | 安裝方式 | 用途 |
|-----|---------|------|
| [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) | `npm install -g @anthropic-ai/claude-code` | AI 程式代理執行環境 |
| [OpenSpec CLI](https://github.com/openspec-dev/openspec) | `npm install -g openspec` | 規格對齊引擎（proposal → specs → design → tasks） |
| [GSD skill](https://github.com/wmayner/gsd) | 在 Claude Code 中執行 `/skill-install gsd` | 執行引擎（plan → execute → verify → ship） |

##### 安裝步驟

```bash
# 1. Clone D-Team 並複製到你的專案
git clone https://github.com/anthropics/D-Team.git d-team-source
cp -r d-team-source/.claude/ /path/to/your-project/.claude/
cp d-team-source/CLAUDE.md /path/to/your-project/CLAUDE.md
rm -rf d-team-source

# 2. 初始化 OpenSpec
cd /path/to/your-project
openspec init
# 編輯 openspec/config.yaml — 填入專案名稱、描述、約束條件

# 3. 驗證安裝
claude
# 輸入：/opsx:explore "test"
# 應該能正常進入探索模式
```

##### GSD 使用注意

GSD 有自己的內部 agents（`gsd-planner`、`gsd-executor` 等），負責**執行編排**。D-Team 的 agents 負責**領域工作**（寫程式、審查、測試）。兩者分工：

| 層級 | 負責方 | 範例 |
|------|-------|------|
| **執行編排** | GSD 內部 agents | 任務規劃、波次平行、狀態追蹤、原子提交 |
| **領域工作** | D-Team agents | 寫程式、程式碼審查、安全稽核、測試、文件 |

**重要**：D-Team 流程中 GSD 的 `/gsd:discuss-phase` 會被**跳過**，因為 OpenSpec specs 已經涵蓋了「要做什麼」的討論。流程改為：

```
/opsx:propose → （確認 specs）→ /gsd:plan-phase → /gsd:execute-phase
```

##### 常用指令

```bash
# OpenSpec 指令（規格對齊）
/opsx:explore "主題"               # 探索一個問題
/opsx:propose "功能名稱"            # 建立 proposal + specs + design
/opsx:verify "change-name"         # 驗證實作是否符合規格
/opsx:archive "change-name"        # 歸檔已完成的 change
/opsx:sync "change-name"           # 開發中途同步 delta specs
/opsx:ff "name" "description"      # 一次產生所有規格文件

# GSD 指令（執行管理）
/gsd:quick "描述"                   # 微型修復（< 30 分鐘）
/gsd:fast "描述"                    # 小型功能（30 分鐘 ~ 2 小時）
/gsd:plan-phase N                  # 從 specs 規劃執行階段
/gsd:execute-phase N               # 波次平行執行
/gsd:verify-work N                 # 功能驗證
/gsd:ship N                        # 建立 PR
/gsd:pause-work                    # 儲存 session 狀態
/gsd:resume-work                   # 恢復 session 狀態
/gsd:map-codebase                  # 四向平行 codebase 分析
```

---

#### 方案 B：Codex

##### 必要安裝

| 套件 | 安裝方式 | 用途 |
|-----|---------|------|
| [Codex CLI](https://github.com/openai/codex) | `npm install -g @openai/codex` | AI 程式代理執行環境（OpenAI） |
| [OpenSpec CLI](https://github.com/openspec-dev/openspec) | `npm install -g openspec` | 規格對齊引擎 |

> **注意**：GSD 是 Claude Code 專屬的 skill，**Codex 無法使用**。Codex 的工作流程以 OpenSpec 做規格對齊，由 Tech Lead agent 直接分解任務並協調 agents 執行。

##### 安裝步驟

```bash
# 1. Clone D-Team 並複製 Codex 所需檔案
git clone https://github.com/anthropics/D-Team.git d-team-source
cp -r d-team-source/agents/ /path/to/your-project/agents/
cp -r d-team-source/.codex/ /path/to/your-project/.codex/
cp d-team-source/AGENTS.md /path/to/your-project/AGENTS.md
cp -r d-team-source/.claude/skills/ /path/to/your-project/.agents/skills/
rm -rf d-team-source

# 2. 初始化 OpenSpec
cd /path/to/your-project
openspec init

# 3. 驗證安裝
codex
# 問：「列出可用的 agents」
# 應該顯示全部 14 個 agents
```

##### Codex vs Claude Code 差異

| 功能 | Claude Code | Codex |
|-----|------------|-------|
| Agent 調度 | Task tool（subagent 模式） | Multi-agent threads（`.codex/config.toml`） |
| GSD 執行 | `/gsd:*` 指令，波次平行 | 不支援 — Tech Lead 手動協調 |
| OpenSpec skills | `/opsx:*` slash commands | 手動讀取 SKILL.md 並執行步驟 |
| Session 持久化 | `/gsd:pause-work` / `resume-work` | 手動撰寫交接筆記 |
| 模型選擇 | `opus` / `sonnet` / `haiku` | 在 `.toml` 檔案設定 `model` |
| 審查 agents | Task tool 中唯讀 | `.toml` 設定 `sandbox_mode = "read-only"` |

##### Codex 工作流程（無 GSD）

```
1. 探索       → 讀取 openspec-explore skill，調查 codebase
2. 提案       → openspec new change "name" → 建立規格文件
3. 分解       → Tech Lead 從 design.md 分解任務
4. 執行       → 委派給 backend/frontend engineers
5. 審查       → code-reviewer（Fix-first）+ security-reviewer
6. 驗證       → 讀取 openspec-verify skill，執行驗證
7. 交付       → 建立 PR，歸檔 change
```

---

### 專案起手式

#### 全新專案（Greenfield）

```
Day 0 — 初始化
│
├─ 1. 建立專案 repo，安裝基礎依賴
├─ 2. 安裝 D-Team（參考上方「環境建置」）
├─ 3. 初始化 OpenSpec
│     $ openspec init
│     編輯 openspec/config.yaml：
│       - 專案名稱與描述
│       - 技術棧約束（語言、框架、資料庫）
│       - 團隊慣例（命名風格、commit 格式）
│
├─ 4. 建立技術棧 skills（參考「建立技術棧 Skill」章節）
│     .claude/skills/your-language-patterns/SKILL.md
│     .claude/skills/your-framework-patterns/SKILL.md
│
├─ 5. 將 skills 連結到 agents
│     更新 backend-engineer.md、frontend-engineer.md、code-reviewer.md
│
└─ 6. 第一個功能
      $ claude
      /opsx:explore "這個專案要解決什麼問題"
      /opsx:propose "initial-setup"
      → Architect 產出 design.md（專案結構、API 契約、資料模型）
      → 使用者確認
      /gsd:plan-phase 1
      /gsd:execute-phase 1
```

**Greenfield 建議：**
- 花時間寫好 `openspec/config.yaml` — 這是專案的 DNA，所有 agents 都會讀取它來理解專案背景
- 第一次 `/opsx:propose` 應該涵蓋專案骨架：目錄結構、build 工具、CI pipeline、基礎設定檔
- 讓 Architect 盡早定義基礎模式 — 後續所有程式碼都會繼承這些決策
- 初始建置完成後執行 `/opsx:archive`，把基礎 specs 鎖定為專案的長期參考

#### 既有專案（Brownfield）

```
Day 0 — 導入
│
├─ 1. 安裝 D-Team 到既有專案（參考上方「環境建置」）
│
├─ 2. 初始化 OpenSpec，填入既有背景
│     $ openspec init
│     編輯 openspec/config.yaml：
│       - 描述既有架構、技術棧、團隊慣例
│       - 標註已知的技術債或特殊約束
│       - 參照既有的架構文件（如果有的話）
│
├─ 3. 掃描 codebase
│     $ claude
│     /gsd:map-codebase
│     → 產出四向分析：技術棧、架構、品質、風險
│     → 讀取輸出，了解 D-Team 面對的是什麼
│
├─ 4. 從既有模式建立技術棧 skills
│     讀你的 codebase，萃取團隊已經在遵循的模式
│     把它們寫成 skill 檔案，讓 agents 符合你的慣例
│
├─ 5. 補建基礎 specs（建議但非必要）
│     /opsx:explore "目前的系統架構"
│     → 把既有功能記錄成 specs 放在 openspec/specs/
│     → 讓 agents 知道什麼東西已經存在
│
└─ 6. 第一次改動
      /opsx:propose "your-first-change"
      → Agents 會讀取既有程式碼 + specs 來理解邊界
      → Architect 的 design.md 會尊重既有模式
      → 接下來走正常流程
```

**Brownfield 建議：**
- `/gsd:map-codebase` 是必要步驟 — 沒有它，agents 可能會建立重複功能或違反既有模式
- 建立的 skills 應反映你專案「目前的」慣例，不是理想化的慣例
- 先從一個小範圍、明確的改動開始，驗證流程順暢後再處理大功能
- 如果專案有架構文件，在 `openspec/config.yaml` 中參照它們，讓 agents 繼承那些背景知識
- 既有測試是你的安全網 — 確認 `tdd-enforcement.md` 有正確的測試指令後再讓 agents 動手寫程式

---

### 開發流程

#### 完整生命週期（中型功能，Claude Code）

```
┌─────────────────────────────────────────────────────────────────┐
│                          開 發 生 命 週 期                        │
└─────────────────────────────────────────────────────────────────┘

Phase 0: 探索（選擇性）
┌──────────────────────────────────────────────┐
│  /opsx:explore "問題描述"                      │
│                                               │
│  → 調查 codebase                              │
│  → ASCII 圖表、權衡分析                         │
│  → 發掘風險與未知                               │
│  → 不寫程式 — 純粹思考                          │
│                                               │
│  產出：對問題的共識                              │
└──────────────────┬───────────────────────────┘
                   ▼
Phase 1: 提案（規格對齊）
┌──────────────────────────────────────────────┐
│  /opsx:propose "feature-name"                 │
│                                               │
│  PM 撰寫：                                    │
│    ├─ proposal.md     （做什麼、為什麼）         │
│    └─ specs/          （詳細需求規格）           │
│  Architect 撰寫：                              │
│    └─ design.md       （怎麼做 — 契約、         │
│                        資料模型、架構圖）         │
│                                               │
│  ⚠ 使用者確認 specs + design 後才進入下一階段     │
│                                               │
│  產出：openspec/changes/<name>/                │
└──────────────────┬───────────────────────────┘
                   ▼
Phase 2: 規劃（執行計畫）
┌──────────────────────────────────────────────┐
│  /gsd:plan-phase N                            │
│                                               │
│  GSD 讀取 OpenSpec specs + design              │
│  產出任務計畫，每個任務包含：                      │
│    ├─ <files>  涉及的檔案                       │
│    ├─ <verify> 驗證標準                         │
│    └─ <done>   完成條件                         │
│  計畫品質驗證（最多迭代 3 次）                     │
│                                               │
│  產出：.planning/phases/XX-YY-PLAN.md          │
└──────────────────┬───────────────────────────┘
                   ▼
Phase 3: 執行（實作）
┌──────────────────────────────────────────────┐
│  /gsd:execute-phase N                         │
│                                               │
│  波次平行執行：                                 │
│    Wave 1: [任務A] [任務B] [任務C]  ← 平行     │
│    Wave 2: [任務D] [任務E]          ← 平行     │
│    Wave 3: [任務F]                             │
│                                               │
│  每個任務：                                     │
│    ├─ 獨立 200K context window                 │
│    ├─ TDD：寫測試 → 實作 → 重構                 │
│    ├─ 每次修復前檢查 WTF-likelihood              │
│    └─ 完成後產出原子 git commit                  │
│                                               │
│  接著：code-reviewer（Fix-first 流程）           │
│        security-reviewer（如涉及 auth/API）      │
│                                               │
│  產出：程式碼 + 測試 + 審查結果                   │
└──────────────────┬───────────────────────────┘
                   ▼
Phase 4: 驗證（雙重驗證）
┌──────────────────────────────────────────────┐
│  /gsd:verify-work N                           │
│    → 功能面：所有 <done> 條件都達成了嗎？         │
│    → 測試通過了嗎？                              │
│                                               │
│  /opsx:verify <change-name>                   │
│    → 完整性：所有需求都有對應實作嗎？              │
│    → 正確性：行為符合 Given/When/Then 場景嗎？    │
│    → 一致性：設計決策都被遵循了嗎？                │
│                                               │
│  ⚠ 兩者都必須通過才能繼續                         │
│                                               │
│  產出：驗證報告                                  │
└──────────────────┬───────────────────────────┘
                   ▼
Phase 5: QA（測試）
┌──────────────────────────────────────────────┐
│  qa-engineer                                  │
│    → 從 OpenSpec 場景撰寫測試案例                │
│    → 驗證邊界案例和錯誤路徑                       │
│                                               │
│  e2e-runner                                   │
│    → 執行關鍵使用者流程測試                       │
│    → 診斷失敗原因（測試問題 vs 應用問題）           │
│                                               │
│  如果 REJECTED → 帶著缺陷報告回到 Phase 3        │
│                                               │
│  產出：QA 報告（APPROVED / REJECTED）            │
└──────────────────┬───────────────────────────┘
                   ▼
Phase 6: 發布
┌──────────────────────────────────────────────┐
│  doc-updater     → 更新架構文件                  │
│  product-manager → 更新使用者文件                 │
│  process-reviewer → 流程回顧報告                 │
│                                               │
│  /opsx:archive <change-name>                  │
│    → 移至 archive/，合併 delta specs             │
│                                               │
│  /gsd:ship N                                  │
│    → 建立 PR（自動產生摘要）                      │
│                                               │
│  產出：可合併的 PR                               │
└──────────────────────────────────────────────┘
```

#### 不同規模的簡化版流程

**微型修復（< 30 分鐘）：**
```
/gsd:quick "修復 bug"  →  完成
```

**小型功能（30 分鐘 ~ 2 小時）：**
```
/gsd:fast "加入端點"  →  code-reviewer  →  完成
  或
/opsx:propose "功能"  →  手動實作  →  /opsx:archive
```

**大型功能（> 1 天）：**
```
/opsx:explore  →  /opsx:propose  →  /gsd:plan-phase 1..N
  →  每個 phase 重複：
       /gsd:execute-phase  →  /gsd:verify-work  →  /opsx:verify
  →  /opsx:archive  →  /gsd:ship
  →  跨 session 使用 /gsd:pause-work + /gsd:resume-work
```

#### 異常處理

| 狀況 | 觸發條件 | 後續處理 |
|------|---------|---------|
| 修復風險太高 | WTF-likelihood >20% | Agent 停止，呈現風險評估，詢問使用者 |
| 連續失敗 | 3 次假設失敗 | Agent 停止，呈現嘗試過的方案，等待指示 |
| 修改範圍太廣 | 爆炸半徑 >5 個檔案 | Agent 停止，請使用者確認範圍 |
| 審查發現嚴重問題 | CRITICAL 等級 | 審查結論 = BLOCK，修復後才能合併 |
| 規格驗證未通過 | 缺少需求實作 | 回到 Phase 3 補上缺漏 |
| QA 退回 | CRITICAL 測試失敗 | 帶著缺陷報告回到 Phase 3 |
| 建構失敗 | 編譯/型別錯誤 | 自動呼叫 `build-resolver` |
| 工作中斷 | 使用者需要離開 | `/gsd:pause-work` 儲存完整狀態 |
| GSD 計畫與規格衝突 | 偵測到矛盾 | 規格優先 — 修正計畫，不改規格 |

---

### 依功能規模選擇流程

| 規模 | 工時 | Claude Code 流程 | Codex 流程 |
|------|------|-----------------|-----------|
| **微型** | < 30 分鐘 | `/gsd:quick "修復"` | 直接改程式 |
| **小型** | 30 分鐘 ~ 2 小時 | `/gsd:fast "功能"` | OpenSpec propose + 手動實作 |
| **中型** | 2 ~ 8 小時 | OpenSpec + GSD 完整流程 | OpenSpec + Tech Lead 協調 |
| **大型** | > 1 天 | 多階段 OpenSpec + GSD | 多 session + 交接筆記 |

### 團隊 Agents（14 個）

| 群組 | Agent | Claude 模型 | Codex Sandbox | 職責 |
|------|-------|------------|---------------|------|
| 協調者 | `tech-lead` | opus | workspace-write | 流程編排、工作流管理 |
| 規格 | `product-manager` | sonnet | workspace-write | 需求、proposal、specs |
| 規格 | `architect` | opus | workspace-write | 系統設計、技術決策 |
| 核心 | `backend-engineer` | sonnet | workspace-write | 後端實作 |
| 核心 | `frontend-engineer` | sonnet | workspace-write | 前端實作 |
| 核心 | `devops-engineer` | sonnet | workspace-write | 基礎設施、CI/CD |
| 品質 | `code-reviewer` | opus | read-only | Fix-first 程式碼審查 |
| 品質 | `security-reviewer` | sonnet | read-only | 安全稽核 |
| 品質 | `qa-engineer` | sonnet | read-only | 測試案例、UAT |
| 品質 | `process-reviewer` | sonnet | read-only | 流程回顧 |
| 工具 | `build-resolver` | haiku | workspace-write | 建構錯誤修復 |
| 工具 | `e2e-runner` | haiku | workspace-write | E2E 測試執行 |
| 工具 | `refactor-cleaner` | haiku | workspace-write | 死碼清理 |
| 工具 | `doc-updater` | haiku | workspace-write | 文件更新 |

### 安全機制

#### WTF-Likelihood 停損（`rules/wtf-likelihood.md`）

防止執行類 agents 越修越糟：
- **>20% 風險閾值** — 修復風險太高時停止
- **50 次修復硬上限** — 單次 session 最多修復次數
- **三振出局** — 連續 3 次假設失敗後暫停
- **5 檔案爆炸半徑** — 修復範圍超過 5 個檔案需先確認

#### Fix-First 審查（`rules/fix-first-review.md`）

程式碼審查者區分機械式修復與判斷型問題：
- **自動修復**（不需使用者同意）：死碼、過期註解、magic numbers、N+1 查詢
- **詢問使用者**（需要同意）：安全決策、race condition、設計選擇
- **30 分鐘門檻**：修復成本超過 30 分鐘的標記為技術債，不阻擋合併

### 建立技術棧 Skill

D-Team 出廠時不帶任何語言或框架的知識。你需要建立 **skill 檔案**來注入技術棧的專業知識——這些是 agents 在工作前會載入的可重用指令集。

#### Skill 檔案格式

每個 skill 獨立一個資料夾，內含 `SKILL.md`：

```
.claude/skills/{skill-name}/SKILL.md       ← Claude Code
.agents/skills/{skill-name}/SKILL.md       ← Codex（自動偵測）
```

檔案結構：

```markdown
---
name: {Skill 名稱}
description: {一句話描述 — 什麼時候載入這個 skill}
---

# {Skill 名稱}

## When to Use
{Agent 應該載入這個 skill 的條件}

## Patterns
### {模式分類 1}
{具體的程式碼模式、慣例、正確/錯誤範例}

## Commands
{與此技術棧相關的 build、test、lint、format 指令}

## Anti-Patterns
{常見錯誤寫法，附帶原因說明}
```

#### Skill 裡該放什麼

| 內容類型 | 適合放在 Skill | 不適合（用 Rules） |
|---------|--------------|------------------|
| 語言慣用寫法 | 適合 | — |
| 框架專屬模式 | 適合 | — |
| Build/test/lint 指令 | 適合 | — |
| 函式庫使用模式 | 適合 | — |
| 反模式與原因 | 適合 | — |
| 流程約束（誰做什麼） | — | 放 `rules/` |
| 品質閘門（覆蓋率門檻） | — | 放 `rules/` |
| 工作流程序列 | — | 放 `rules/` |

#### 完整步驟：加入新技術棧

**1. 建立 skill 檔案** — 例如 `.claude/skills/golang-patterns/SKILL.md` 和 `.claude/skills/postgres-patterns/SKILL.md`

**2. 連結 skill 到 agents** — 在 `backend-engineer.md`、`code-reviewer.md` 等加入 `## Required Skills` 區段

**3. 更新 TDD 指令** — 在 `rules/tdd-enforcement.md` 中設定你的技術棧的測試/覆蓋率指令

**4.（選擇性）加入專業 reviewer** — 例如 `database-reviewer.md`

**5. Codex 同步** — 將 skills 複製到 `.agents/skills/` 並更新 `AGENTS.md`

#### Skill 對照表：哪些 Agents 需要哪些 Skills

| Skill 類別 | 連結到這些 Agents |
|-----------|-----------------|
| 後端語言（Go、Python、Rust 等） | `backend-engineer`、`code-reviewer` |
| 前端框架（React、Vue、Svelte 等） | `frontend-engineer`、`code-reviewer` |
| 資料庫（PostgreSQL、MongoDB 等） | `backend-engineer`、`database-reviewer`（需額外建立） |
| 基礎設施（Docker、K8s、Terraform 等） | `devops-engineer` |
| 安全（OWASP、認證模式） | `security-reviewer` |
| 測試框架（Jest、pytest 等） | `qa-engineer`、`e2e-runner` |

#### 各技術棧速查

| 技術棧 | 要建立的 Skills | 要更新的 Agents |
|--------|----------------|----------------|
| **Go + PostgreSQL** | `golang-patterns`、`postgres-patterns` | backend-engineer、code-reviewer、+database-reviewer |
| **TypeScript + MongoDB** | `typescript-patterns`、`mongodb-patterns` | backend-engineer、code-reviewer、+database-reviewer |
| **Python + FastAPI** | `python-patterns`、`fastapi-patterns` | backend-engineer、code-reviewer |
| **React / Next.js** | `react-patterns`、`nextjs-patterns` | frontend-engineer、code-reviewer |
| **Rust + SQLite** | `rust-patterns`、`sqlite-patterns` | backend-engineer、code-reviewer |
| **Vue + Supabase** | `vue-patterns`、`supabase-patterns` | frontend-engineer、code-reviewer |

### 目錄結構

#### Claude Code 配置

```
your-project/
├── CLAUDE.md                          ← 團隊全局指令
├── openspec/                          ← OpenSpec 規格文件
│   ├── config.yaml
│   ├── specs/
│   └── changes/
├── .planning/                         ← GSD 執行狀態
├── .worklog/                          ← 階段性工作紀錄
└── .claude/
    ├── agents/
    ├── skills/
    └── rules/
```

#### Codex 配置

```
your-project/
├── AGENTS.md                          ← Codex 團隊指令
├── openspec/                          ← OpenSpec 規格文件（共用）
├── .worklog/                          ← 階段性工作紀錄（共用）
├── agents/                            ← Codex runtime configs（.toml）
│   ├── tech-lead.toml
│   ├── core/
│   ├── quality/
│   └── tools/
├── .codex/
│   ├── config.toml                    ← 多代理設定
│   ├── agents/                        ← Agent playbooks（.md）
│   └── rules/                         ← Codex 規則
└── .agents/
    └── skills/                        ← 共用 skills（自動偵測）
```

#### 雙平台配置（Claude Code + Codex 並用）

```
your-project/
├── CLAUDE.md                          ← Claude Code 進入點
├── AGENTS.md                          ← Codex 進入點
├── openspec/                          ← 共用
├── .worklog/                          ← 共用
├── .claude/                           ← Claude Code 設定
├── .codex/                            ← Codex 設定
├── agents/                            ← Codex runtime（.toml）
└── .agents/skills/                    ← Codex skill 偵測
```

---

## License

MIT
