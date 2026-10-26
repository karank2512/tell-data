---
report: ops
title: "Operations and resolution"
as_of: 2026-10-26
generated_at: 2026-10-26T07:04:08+00:00
n: 618
quotable: true
flagged_numbers: 5
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

Cohort N: 618.

## Numbers

### Unattended operation

The unattended period starts at the last run marked manual (2026-10-21T01:25:43.069721+00:00).

- Weeks unattended (days since the last manual intervention needed to keep ingest green, divided by 7): 0.7 weeks
- Weeks since collection started (period covered by the report): 1.7 weeks
- Dead-man proxy (tell runlog --week): weeks since the last 24-hour gap: 0.0 weeks

### Reliability and freshness

- Success rate ok / (ok + failed), percent, overall: 96.8% (N=618)
- Freshness p95, hours (lag from the earliest moment tell could have seen the event to the moment it was recorded; first looks, first-seen-only events and backfills excluded): 60.8 h (N=89)
- Collector events observed in the 24-hour freshness window: 286
- Left out of freshness, first looks (tell was not yet watching the board, company feed, Google News query or source): 0
- Left out of freshness, first-seen-only (no source timestamp: tell dated the event when it saw it): 197
- Left out of freshness, backfills (archive loads and widened Google News windows): 0

| source | finished runs | ok | success rate |
| --- | --- | --- | --- |
| ashby | 45 | 45 | 100.0% (N=45) |
| discovery | 44 | 44 | 100.0% (N=44) |
| edgar | 50 | 45 | 90.0% (N=50) |
| greenhouse | 45 | 45 | 100.0% (N=45) |
| issuer_discovery | 44 | 44 | 100.0% (N=44) |
| lever | 45 | 45 | 100.0% (N=45) |
| news | 49 | 34 | 69.4% (N=49) |
| prune | 47 | 47 | 100.0% (N=47) |
| publish | 94 | 94 | 100.0% (N=94) |
| resolve | 50 | 50 | 100.0% (N=50) |
| score | 49 | 49 | 100.0% (N=49) |
| slack_digest | 10 | 10 | 100.0% (N=10; too small to quote) |
| slack_nudge | 1 | 1 | 100.0% (N=1; too small to quote) |
| smartrecruiters | 45 | 45 | 100.0% (N=45) |

### Freshness by source

The same 24 hours, per source: events observed, events counted, events left out and the p95 of tell's lag (ADR 0043).

| source | events observed | counted | first looks | first-seen-only | backfills | freshness p95 |
| --- | --- | --- | --- | --- | --- | --- |
| ashby | 116 | 9 | 0 | 107 | 0 | 23.3 h (N=9; too small to quote) |
| greenhouse | 90 | 8 | 0 | 82 | 0 | 21.0 h (N=8; too small to quote) |
| lever | 8 | 0 | 0 | 8 | 0 | n/a (N=0; too small to quote) |
| news | 72 | 72 | 0 | 0 | 0 | 60.8 h (N=72) |

### Coverage

- Companies monitored (active board or a CIK): 965
- Postings tracked (closed_at is null): 11,587

### Resolution

Gold set measured 2026-10-05, from docs/methods/entity-resolution.md.

- Review-queue items per 1,000 events, trailing 28 days: 27.94 (N=49,348)
- Review-queue items created in the period: 1,379

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

As of 2026-10-26; generated 2026-10-26T07:04:08+00:00.
