---
report: velocity-lead
title: "Hiring velocity as a leading indicator of a Form D filing"
as_of: 2026-10-19
generated_at: 2026-10-19T07:02:24+00:00
n: 0
quotable: false
flagged_numbers: 7
script: tell/backtest/velocity_lead.py
private: false
---

# Hiring velocity as a leading indicator of a Form D filing

Dated report written by `tell backtest velocity-lead` (spec section 13.2). The prospective study is the real result once it is runnable; the two retrospective reconstructions are rougher and are labelled as such, each with its coverage.

**Not quotable:** N=0 is under 30; not yet runnable, 5 days of collection.

## Question

Does a hiring spike precede a funding filing more often for companies that file than for comparable companies that do not, and by how many days?

## Cohort

Prospective: every new Form D (isAmendment = false) filed after day 60 of collection by a company with an active board, against non-filers with an active board matched by open-posting decile. Retrospective: new Form D filings since 2025-10-01 (database events plus quarterly data sets) by companies with an active board. Cohort N is the prospective cohort.

Cohort N: 0.

**Not yet runnable: 5 days of collection < 60.**

## Numbers

### Prospective (the result to quote)

Not yet runnable: 5 days of collection < 60. The retrospective versions below are rougher and are never quoted as the result.

- Days of collection: 5

### Retrospective (a): current postings' posted_at (survivorship bias)

Filings: 364 from database events, 0 from 0 quarterly data sets; 5 by companies with an active board. The difference between filers and controls is the number to quote, with both N.

- Retrospective (a) coverage: filings with a reconstructed posting in the 12 weeks before: 80.0% (N=5; too small to quote)
- Postings carrying a provider posted_at: 100.0% (N=10,588)
- Retrospective (a): filings in the cohort: 5
- Retrospective (a): share of filings preceded by a spike in the 8 weeks before: 60.0% (N=5; too small to quote)
- Retrospective (a): median lead, days from the first spike week to the filing: 56.0 days (N=3; too small to quote)
- Retrospective (a): control share preceded by a spike (matched by open-posting decile): 44.2% (N=206)
- Retrospective (a): difference, filers minus controls, percentage points: 15.79 (N=5; too small to quote)

### Retrospective (b): Wayback Machine snapshots of the board page

Of 5 filings, 0 have a Greenhouse or Lever board; 0 have a CDX listing in the cache (0 not fetched yet), 0 have a snapshot in the 12 weeks and 0 are covered (a snapshot in at least 8 of the 12 weeks).

- Retrospective (b) coverage: covered filings of those with a supported board: n/a (N=0; too small to quote)
- Retrospective (b): filings in the cohort: 0
- Retrospective (b): share of filings preceded by a spike in the 8 weeks before: n/a (N=0; too small to quote)
- Retrospective (b): median lead, days from the first spike week to the filing: n/a (N=0; too small to quote)

## Query or script

Script: `tell/backtest/velocity_lead.py`.

Spike rule (company level, all families):

```text
week k before filing F: F - 7k days < opened_at <= F - 7(k - 1) days, k = 1..12
new7(k) = net-new postings opened in week k (no interns, no reposts,
          same fingerprint on the same board within 30 days = one)
baseline = max(mean(new7(9..12)), 0.5)
spike    = any k in 1..8 with new7(k) >= 3 and new7(k) / baseline >= 1.5
lead     = 7 * max(spiking k) days
```

## Caveats

- Company level: all role families are counted together; tell's live hiring_velocity signal is per family, so this is a coarser rule than the one that alerts.
- Filings between day 60 and day 84 of collection have a baseline that reaches back before collection started (floored at 0.5), which makes a spike easier to reach.
- Retrospective (a) only sees postings that were still open when tell started watching or opened since: closed roles are invisible, so spikes are undercounted.
- Retrospective (b) dates an opening at the snapshot that first shows it; a gap between snapshots lumps several weeks of openings into one week. Snapshots exist for a minority of boards, and Ashby and SmartRecruiters pages list no jobs in their HTML.
- The control group is matched by open-posting decile at the median filing date; companies that file without telling the SEC (or file under another name tell did not link) sit in the control group.
- Once the prospective study is runnable, it is the result; the retrospective numbers are context only.

## Date

As of 2026-10-19; generated 2026-10-19T07:02:24+00:00.
