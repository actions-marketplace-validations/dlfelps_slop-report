# Slop Report

**Automatic PR quality reports for Python projects.**

Slop Report posts a code quality summary as a comment on every pull request — coverage, blast radius, performance regressions, and maintainability — so reviewers have the data they need without leaving GitHub.

| Metric | Score | Status | Details |
|--------|-------|--------|---------|
| Change Risk | 72% covered | ⚠️ | 28% of changed lines lack test coverage |
| Blast Radius | 12 modules | 📉 | High impact: auth, api, models affected |
| Performance | No regressions | ✅ | No tests exceeded 20% slowdown threshold |
| Maintainability | 74 / 100 | ✅ | Above threshold (65) |

Runs alongside your existing CI — it never blocks a merge, just surfaces the signal.

---

## Quick Start

```yaml
# .github/workflows/slop-report.yml
name: Slop Report

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  pull-requests: write
  contents: read

jobs:
  slop-report:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0   # required — Slop Report diffs against the base branch

      - name: Run Slop Report
        uses: dlfelps/slop-report@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

---

## Metrics

| Metric | What it measures |
|--------|-----------------|
| **Change Risk** | % of added/modified lines covered by your existing test suite |
| **Blast Radius** | How many other modules import the files you changed |
| **Performance** | Per-test timing compared to the base branch — regressions are flagged |
| **Maintainability** | Radon Maintainability Index for changed files (0–100) |

---

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `github-token` | `${{ github.token }}` | Token for posting the PR comment |
| `enable-coverage` | `true` | Run change risk / coverage analysis |
| `enable-blast-radius` | `true` | Run blast radius / dependency analysis |
| `enable-performance` | `true` | Run performance regression analysis |
| `enable-maintainability` | `true` | Run maintainability index analysis |
| `coverage-threshold` | `80` | Minimum % of changed lines covered (0–100) |
| `maintainability-threshold` | `65` | Minimum Radon MI score (0–100) |
| `performance-threshold` | `20` | Max % slowdown before flagging a regression |

---

## Configuration Examples

### Disable a metric

```yaml
- uses: dlfelps/slop-report@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    enable-performance: 'false'
```

### Stricter thresholds

```yaml
- uses: dlfelps/slop-report@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    coverage-threshold: '90'
    maintainability-threshold: '75'
    performance-threshold: '10'
```

---

## How It Works

### Change Risk
Runs your test suite via `coverage run -m pytest` (auto-detected; falls back to `unittest`), then cross-references coverage data against the exact lines added or modified in the PR. Reports the percentage of those lines actually exercised by tests.

### Blast Radius
Parses every Python file in the repository with Python's `ast` module to build an import graph. Identifies all files that import the modules you changed, and classifies impact as Low (≤3), Medium (4–10), or High (>10 affected modules).

### Performance
Runs `pytest --durations=0` on both the base branch and the PR branch using `git archive`, then compares per-test timings. Tests that slowed by more than the configured threshold are flagged, with the worst offender highlighted in the report.

### Maintainability
Runs `radon mi` on the changed files and reports the average Maintainability Index score (0–100). Scores ≥ threshold are ✅, within 10 points below are ⚠️, and further below are 📉.

---

## Requirements

- Python project with pytest or unittest
- `fetch-depth: 0` on the `actions/checkout` step
- `pull-requests: write` permission for posting the comment

---

## Status Indicators

| Icon | Meaning |
|------|---------|
| ✅ | Metric is within acceptable range |
| ⚠️ | Metric is below threshold or needs attention |
| 📉 | Significant issue detected |
