---
name: Retro
description: Sprint retrospective analyzing git metrics, shipping velocity, and collaboration patterns with trend tracking
---

# Retro

Analyze the team's shipping velocity, code health, and collaboration patterns using git history as the primary data source. Produce actionable insights, not just numbers.

## Invocation

- `/retro` — Analyze the last 7 days (default)
- `/retro 14d` — Analyze the last 14 days
- `/retro 30d` — Analyze the last 30 days
- `/retro compare` — Side-by-side comparison with prior same-length window

## Data Collection

### Git History Extraction

```bash
# Extract commits for the analysis window
git log --since="N days ago" --format="%H|%an|%ae|%aI|%s" --numstat
```

Collect:
- Commit hashes, authors, timestamps, messages
- Files changed, lines added/removed per commit
- Branch/merge patterns

### Metrics Computed

#### Shipping Velocity
| Metric | How |
|--------|-----|
| Total commits | Count in window |
| LOC added / removed | Sum from numstat |
| Net LOC change | Added minus removed |
| Commit frequency | Commits per day |
| Active days | Days with at least one commit |

#### Code Health
| Metric | How |
|--------|-----|
| Test ratio | Test file changes / total file changes |
| Churn hotspots | Files changed 3+ times in the window |
| Commit type distribution | Categorize by conventional commit prefix (feat/fix/refactor/test/chore/docs) |
| Large commit detection | Commits touching >10 files or >500 LOC |

#### Work Patterns
| Metric | How |
|--------|-----|
| Hourly distribution | Histogram of commit hours (local timezone) |
| Session detection | Group commits by 45-minute gap threshold |
| Session types | Deep (>2hr), Medium (30min-2hr), Micro (<30min) |
| Focus ratio | Time in deep sessions / total session time |

#### Collaboration (if multi-contributor)
| Metric | How |
|--------|-----|
| Per-contributor commits | Breakdown by author |
| Review turnaround | Time from PR open to merge (if using PRs) |
| Co-authorship | `Co-Authored-By` frequency |

## Analysis

### For Each Metric, Provide:
1. **The number** — Raw value
2. **The context** — Is this typical? Higher/lower than expected?
3. **The insight** — What does this mean for the team?

### Specific Praise (Evidence-Based)
For each contributor, identify one specific accomplishment anchored in actual commits:
- "Shipped the checkout refactor (12 commits, 3 days) with 94% test coverage"
- NOT "Great work this sprint" (vague, no evidence)

### Growth Opportunities
Identify patterns that suggest improvement areas:
- High churn on specific files → consider better upfront design
- Low test ratio in a subsystem → TDD enforcement gap
- Many micro sessions → context switching may be hurting focus
- Large commits → consider smaller, atomic changes

## Trend Tracking

Save retrospective data to `.retros/` for historical comparison:

```
.retros/
  └── {YYYY-MM-DD}-{window}.json
```

When prior data exists, include trend analysis:
- Velocity trending up/down/stable
- Test ratio improving or declining
- Churn hotspot persistence (same files churning across sprints)

## Compare Mode

When invoked with `/retro compare`:

```
## Side-by-Side Comparison

| Metric | Previous [dates] | Current [dates] | Delta |
|--------|-------------------|------------------|-------|
| Commits | 45 | 52 | +16% |
| Test ratio | 0.28 | 0.35 | +25% |
| Deep sessions | 8 | 12 | +50% |
```

Highlight the most significant changes with interpretation.

## Output Format

```
# Sprint Retrospective

**Period**: [start] → [end] ([N days])
**Branch**: [default branch]

## Velocity
- Commits: N (N/day)
- LOC: +N / -N (net: +N)
- Active days: N/N

## Code Health
- Test ratio: N% [↑/↓/→ vs last]
- Commit types: feat: N, fix: N, refactor: N, test: N, chore: N, docs: N
- Churn hotspots: [top 3 files with change count]

## Work Patterns
- Sessions: N deep, N medium, N micro
- Focus ratio: N%
- Peak hours: [top 3 hours]

## Highlights
- [Evidence-based praise per contributor]

## Growth Opportunities
- [Pattern-based improvement suggestions]

## Trend
[Comparison with prior period if data exists]
```

## Example

Input: `/retro 7d`

Output: Analyzes last 7 days of git history. Finds: 38 commits, test ratio 0.31 (up from 0.24 last week), 3 churn hotspots in `src/api/checkout.ts` (changed 7 times — suggest stabilization). Focus ratio 62% (12 deep sessions out of 19 total). Recommends: batch checkout changes into fewer, more considered commits.

## Guardrails

- Use git history as the single source of truth — do not estimate or guess
- Every praise must reference specific commits or changes
- Every improvement suggestion must cite a specific pattern in the data
- Save results for trend tracking even when metrics look healthy
- Do not judge productivity by LOC alone — context matters
