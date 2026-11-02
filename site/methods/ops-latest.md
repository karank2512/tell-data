---
report: ops
title: "Operations and resolution"
as_of: 2026-11-02
generated_at: 2026-11-02T06:42:20+00:00
n: 716
quotable: true
flagged_numbers: 6
script: tell/backtest/ops.py
private: false
---

# Operations and resolution

Dated report written by `tell backtest ops` (spec section 13.4): how long the pipeline has run without a hand on it, how often runs succeed, how fresh the data is, what it covers and how well names resolve.

Freshness is tell's lag, from the earliest moment tell could have seen an event to the moment it recorded it (ADR 0043), not the age of what tell reads. Events from tell's first look at a board, company feed, Google News query or source, events without a source timestamp of their own, and backfills are counted beside its N, not in it.

## Question

Does tell run unattended, reliably and fresh, at what coverage, and how well does it resolve names to companies?

## Cohort

Every pipeline runs row up to the evaluation instant (the nightly intake reader's runs are left out); success rate and review-queue rate over the trailing 28 days; freshness over the collector events observed in the 24 hours ending at the evaluation instant, as tell's lag, with first looks, first-seen-only events and backfills left out and counted (ADR 0043). Cohort N is the number of finished runs in the period.

Cohort N: 716.

## Numbers

### Unattended operation

The unattended period starts at the last run marked manual (2026-10-21T01:25:43.069721+00:00).

- Weeks unattended (days since the last manual intervention needed to keep ingest green, divided by 7): 1.7 weeks
- Weeks since collection started (period covered by the report): 2.7 weeks
- Dead-man proxy (tell runlog --week): weeks since the last 24-hour gap: 0.1 weeks

### Reliability and freshness

- Success rate ok / (ok + failed), percent, overall: 96.9% (N=716)
- Freshness p95, hours (lag from the earliest moment tell could have seen the event to the moment it was recorded; first looks, first-seen-only events and backfills excluded): 71.9 h (N=379)
- Collector events observed in the 24-hour freshness window: 513
- Left out of freshness, first looks (tell was not yet watching the board, company feed, Google News query or source): 0
- Left out of freshness, first-seen-only (no source timestamp: tell dated the event when it saw it): 134
- Left out of freshness, backfills (archive loads and widened Google News windows): 0

| source | finished runs | ok | success rate |
| --- | --- | --- | --- |
| ashby | 52 | 52 | 100.0% (N=52) |
| discovery | 51 | 51 | 100.0% (N=51) |
| edgar | 57 | 52 | 91.2% (N=57) |
| greenhouse | 52 | 52 | 100.0% (N=52) |
| issuer_discovery | 51 | 51 | 100.0% (N=51) |
| lever | 52 | 52 | 100.0% (N=52) |
| news | 56 | 39 | 69.6% (N=56) |
| prune | 54 | 54 | 100.0% (N=54) |
| publish | 108 | 108 | 100.0% (N=108) |
| resolve | 57 | 57 | 100.0% (N=57) |
| score | 56 | 56 | 100.0% (N=56) |
| slack_digest | 17 | 17 | 100.0% (N=17; too small to quote) |
| slack_nudge | 1 | 1 | 100.0% (N=1; too small to quote) |
| smartrecruiters | 52 | 52 | 100.0% (N=52) |

### Freshness by source

The same 24 hours, per source: events observed, events counted, events left out and the p95 of tell's lag (ADR 0043).

| source | events observed | counted | first looks | first-seen-only | backfills | freshness p95 |
| --- | --- | --- | --- | --- | --- | --- |
| ashby | 80 | 11 | 0 | 69 | 0 | 22.8 h (N=11; too small to quote) |
| greenhouse | 64 | 6 | 0 | 58 | 0 | 21.4 h (N=6; too small to quote) |
| lever | 8 | 2 | 0 | 6 | 0 | 23.0 h (N=2; too small to quote) |
| news | 360 | 360 | 0 | 0 | 0 | 71.9 h (N=360) |
| smartrecruiters | 1 | 0 | 0 | 1 | 0 | n/a (N=0; too small to quote) |

### Coverage

- Companies monitored (active board or a CIK): 980
- Postings tracked (closed_at is null): 11,577

### Resolution

Gold set measured 2026-10-05, from docs/methods/entity-resolution.md.

- Review-queue items per 1,000 events, trailing 28 days: 24.42 (N=57,533)
- Review-queue items created in the period: 1,405

| gold set, names | precision | recall |
| --- | --- | --- |
| public sample (gold_sample.csv), names | 100.0% (N=50) | 100.0% (N=50) |
| full private set (companies.csv), names | 100.0% (N=300) | 97.3% (N=300) |

## Query or script

Script: `tell/backtest/ops.py`.

## Caveats

- Weeks unattended depends on interventions being marked (runs.notes.manual = true, written for every run under `tell --manual` or TELL_RUN_MANUAL=true, which ingest.yml sets on workflow_dispatch); a fix applied without a marked run is invisible, so the dead-man proxy is printed beside it.
- Success rate counts runs, not boards: one failed vendor leg is one failed run.
- Freshness p95 is a 24-hour window ending at the evaluation instant; a quiet day has a small N. The events left out are counted beside it.
- Freshness is tell's lag, not the age of what tell reads (ADR 0043). It runs from the earliest moment tell could have seen an event (the source's own timestamp; for EDGAR the daily index, filing date + 26 h; moved to tell's previous poll of the same board or Google News query when that poll did not list it) to the moment tell recorded it. Reports written before ADR 0043 took SPEC section 12's observed_at - occurred_at literally and measured the age of company-blog archives read on discovery (23298.1 h on 2026-10-19); do not quote them.
- Sources run on different clocks: boards are polled every run, while Google News reaches the long tail of companies once a week, so news dominates the pooled p95; the table gives each source's own number.
- Company feeds keep no poll history: a feed item dated after tell discovered the feed counts from its own date, so a post that appears days after its date overstates the lag.
- Postings tracked and companies monitored are counts at the instant the report runs, not at the as-of date, when the report is regenerated for a past date.
- Gold-set numbers come from the methods page, measured by the CI test on the gold set; they describe the matcher, not the live link quality.

## Date

As of 2026-11-02; generated 2026-11-02T06:42:20+00:00.
