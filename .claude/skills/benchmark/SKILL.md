---
name: Benchmark
description: Performance regression detection measuring timing, bundle size, and resource metrics with trend tracking
---

# Benchmark

Measure and compare page/application performance metrics. Detect regressions before they reach production. Bundle size is the leading indicator — it is deterministic and does not vary with network conditions.

## Metrics Measured

### Timing Metrics
| Metric | Description |
|--------|-------------|
| TTFB | Time to First Byte |
| FCP | First Contentful Paint |
| LCP | Largest Contentful Paint |
| DOM Interactive | DOM ready for interaction |
| DOM Complete | All resources loaded |
| Full Load | Page fully rendered |

### Bundle Analysis
- Total JavaScript size (raw + gzipped)
- Total CSS size (raw + gzipped)
- Per-chunk breakdown for code-split applications
- Tree-shaking effectiveness (named vs default imports)

### Resource Analysis
- Top 10 slowest resources by load time
- Resource count by type (JS, CSS, images, fonts, API calls)
- Total transfer size
- Unused resource detection (loaded but never referenced)

### Performance Budgets
| Metric | Budget | Regression Threshold |
|--------|--------|---------------------|
| LCP | < 2.5s | > 50% increase OR > 500ms absolute |
| FCP | < 1.8s | > 50% increase OR > 500ms absolute |
| Bundle JS | project-specific | > 25% increase |
| Bundle CSS | project-specific | > 25% increase |
| Total requests | project-specific | > 30% increase |

## Workflow

### 1. Baseline Capture

```bash
# Capture current baseline (run on main branch)
/benchmark --baseline
```

Read the application's build output and deployment artifacts. Record:
- Bundle sizes from build manifest or webpack/vite stats
- Route list for multi-page measurement
- Current dependency count and sizes

### 2. Branch Measurement

```bash
# Measure current branch
/benchmark
```

For each route/page:
1. Build the application in production mode
2. Extract bundle sizes from build output
3. If browser available: measure timing metrics via page load
4. Compare against baseline

### 3. Regression Detection

Flag regressions using the threshold table. For each regression:

```
## REGRESSION: [Metric] on [Route/Page]

- Baseline: [value]
- Current: [value]
- Delta: [+N% / +Nms / +NKB]
- Threshold: [which threshold was exceeded]
- Likely Cause: [new dependency / unoptimized import / missing code split]
- Suggested Fix: [specific remediation]
```

### 4. Trend Tracking

Save benchmark results to `.benchmarks/` for historical comparison:
```
.benchmarks/
  └── {YYYY-MM-DD}-{branch}.json
```

Compare against last 5 runs to detect gradual degradation.

## Common Regression Patterns

| Pattern | Detection | Fix |
|---------|-----------|-----|
| Heavy new dependency | Bundle size spike | Replace with lighter alternative or lazy-load |
| Full lodash import | Bundle includes unused utils | Switch to per-function imports or lodash-es |
| Missing code splitting | Single chunk grows | Add dynamic `import()` for route-level splitting |
| Unoptimized images | Large transfer size | Add compression, lazy loading, width/height attrs |
| CSS @import chains | Blocking parallel loads | Use bundler imports instead |
| N+1 API calls | Multiple sequential fetches | Batch or parallelize requests |
| Synchronous scripts | Blocking render | Add async/defer attributes |

## Output Format

```
## Performance Benchmark Report

**Date**: [YYYY-MM-DD]
**Branch**: [branch name]
**Compared Against**: [baseline branch/date]

### Summary
- Regressions: N (X critical, Y warning)
- Improvements: N
- Unchanged: N

### Bundle Size
| Chunk | Baseline | Current | Delta | Status |
|-------|----------|---------|-------|--------|
| main.js | 145KB | 167KB | +15% | ⚠ WARNING |
| vendor.js | 230KB | 230KB | 0% | ✓ OK |

### Timing (if measured)
| Metric | Baseline | Current | Delta | Status |
|--------|----------|---------|-------|--------|
| LCP | 1.2s | 1.8s | +50% | ✗ REGRESSION |

### Regressions Detail
[Per-regression breakdown with cause and fix]

### Verdict: [PASS | WARNING | REGRESSION]
```

## Example

Input: `/benchmark`

Output: Builds the app, extracts bundle stats, compares against last baseline. Finds: main.js grew 22% due to new `moment.js` dependency. Recommends switching to `date-fns` (330KB → 22KB). Saves results to `.benchmarks/`.

## Guardrails

- Never modify application code — measurement only
- Always build in production mode for accurate bundle sizes
- Bundle size is the primary indicator — timing varies with environment
- Save results for trend tracking even if no regressions found
- Report improvements alongside regressions for complete picture
