---
report: ops
title: "Operations and resolution"
as_of: 2026-10-19
generated_at: 2026-10-19T07:02:24+00:00
n: 321
quotable: true
flagged_numbers: 13
script: tell/backtest/ops.py
private: false
---

# Operations and resolution

Dated report written by `tell backtest ops` (spec section 13.4): how long the pipeline has run without a hand on it, how often runs succeed, how fresh the data is, what it covers and how well names resolve.

## Question

Does tell run unattended, reliably and fresh, at what coverage, and how well does it resolve names to companies?

## Cohort

Every pipeline runs row up to the evaluation instant (the nightly intake reader's runs are left out); success rate and review-queue rate over the trailing 28 days; freshness over the last 24 hours of collector events. Cohort N is the number of finished runs in the period.

Cohort N: 321.

## Numbers

### Unattended operation

The unattended period starts at the last run marked manual (2026-10-16T23:16:28.733193+00:00).

- Weeks unattended (days since the last manual intervention needed to keep ingest green, divided by 7): 0.3 weeks
- Weeks since collection started (period covered by the report): 0.7 weeks
- Dead-man proxy (tell runlog --week): weeks since the last 24-hour gap: 0.7 weeks

### Reliability and freshness

- Success rate ok / (ok + failed), percent, overall: 97.8% (N=321)
- Freshness p95, hours (SPEC section 12 definition): 23298.1 h (N=455)

| source | finished runs | ok | success rate |
| --- | --- | --- | --- |
| ashby | 23 | 23 | 100.0% (N=23; too small to quote) |
| discovery | 22 | 22 | 100.0% (N=22; too small to quote) |
| edgar | 27 | 26 | 96.3% (N=27; too small to quote) |
| greenhouse | 23 | 23 | 100.0% (N=23; too small to quote) |
| issuer_discovery | 25 | 25 | 100.0% (N=25; too small to quote) |
| lever | 23 | 23 | 100.0% (N=23; too small to quote) |
| news | 26 | 20 | 76.9% (N=26; too small to quote) |
| prune | 24 | 24 | 100.0% (N=24; too small to quote) |
| publish | 48 | 48 | 100.0% (N=48) |
| resolve | 27 | 27 | 100.0% (N=27; too small to quote) |
| score | 26 | 26 | 100.0% (N=26; too small to quote) |
| slack_digest | 3 | 3 | 100.0% (N=3; too small to quote) |
| slack_nudge | 1 | 1 | 100.0% (N=1; too small to quote) |
| smartrecruiters | 23 | 23 | 100.0% (N=23; too small to quote) |

### Coverage

- Companies monitored (active board or a CIK): 907
- Postings tracked (closed_at is null): 10,585

### Resolution

Gold set measured 2026-10-05, from docs/methods/entity-resolution.md.

- Review-queue items per 1,000 events, trailing 28 days: 35.24 (N=37,709)
- Review-queue items created in the period: 1,329

| gold set, names | precision | recall |
| --- | --- | --- |
| public sample (gold_sample.csv), names | 100.0% (N=50) | 100.0% (N=50) |
| full private set (companies.csv), names | 100.0% (N=300) | 97.3% (N=300) |

## Query or script

Script: `tell/backtest/ops.py`.

## Caveats

- Weeks unattended depends on interventions being marked (runs.notes.manual = true, written for every run under `tell --manual` or TELL_RUN_MANUAL=true, which ingest.yml sets on workflow_dispatch); a fix applied without a marked run is invisible, so the dead-man proxy is printed beside it.
- Success rate counts runs, not boards: one failed vendor leg is one failed run.
- Freshness p95 is a 24-hour window ending at the evaluation instant; a quiet day has a small N.
- Postings tracked and companies monitored are counts at the instant the report runs, not at the as-of date, when the report is regenerated for a past date.
- Gold-set numbers come from the methods page, measured by the CI test on the gold set; they describe the matcher, not the live link quality.

## Date

As of 2026-10-19; generated 2026-10-19T07:02:24+00:00.
