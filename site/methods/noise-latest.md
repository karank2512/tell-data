---
report: noise
title: "Noise handling: reposts, lifetimes and long-open roles"
as_of: 2026-11-02
generated_at: 2026-11-02T06:42:20+00:00
n: 13819
quotable: true
flagged_numbers: 0
script: tell/backtest/noise.py
private: false
---

# Noise handling: reposts, lifetimes and long-open roles

Dated report written by `tell backtest noise` (spec section 13.3): how much of what the boards show is reposts and long-open roles, and how long postings live.

## Question

How much of what job boards show is noise (reposted and long-open roles), and how long does a posting live?

## Cohort

Repost share: postings first seen in the 90 days before the evaluation instant. Lifetime: postings closed by then that carry the provider's posted_at. Long-open share: boards with status active.

Cohort N: 13819.

## Numbers

### Reposts

- Share of postings that are reposts (first seen in the last 90 days), percent: 1.5% (N=13,819)

| vendor | postings | reposts | repost share |
| --- | --- | --- | --- |
| ashby | 7,881 | 126 | 1.6% (N=7,881) |
| greenhouse | 5,203 | 75 | 1.4% (N=5,203) |
| lever | 643 | 2 | 0.3% (N=643) |
| smartrecruiters | 92 | 3 | 3.3% (N=92) |

### Reposts by role family

| family | postings | reposts | repost share |
| --- | --- | --- | --- |
| customer | 1,224 | 6 | 0.5% (N=1,224) |
| data | 159 | 7 | 4.4% (N=159) |
| engineering | 2,904 | 35 | 1.2% (N=2,904) |
| exec | 517 | 4 | 0.8% (N=517) |
| finance_ops | 910 | 17 | 1.9% (N=910) |
| gtm_ops | 781 | 23 | 2.9% (N=781) |
| marketing | 665 | 10 | 1.5% (N=665) |
| ml_ai | 338 | 10 | 3.0% (N=338) |
| other | 3,335 | 32 | 1.0% (N=3,335) |
| product_design | 941 | 22 | 2.3% (N=941) |
| sales | 2,045 | 40 | 2.0% (N=2,045) |

### Posting lifetime

- Median posting lifetime, closed_at - posted_at, days (all families): 55.1 days (N=2,242)

| family | median lifetime |
| --- | --- |
| customer | 50.4 days (N=223) |
| data | 40.5 days (N=32) |
| engineering | 71.3 days (N=372) |
| exec | 64.6 days (N=91) |
| finance_ops | 49.8 days (N=179) |
| gtm_ops | 42.4 days (N=154) |
| marketing | 47.6 days (N=120) |
| ml_ai | 76.8 days (N=50) |
| other | 45.7 days (N=519) |
| product_design | 56.5 days (N=164) |
| sales | 69.0 days (N=338) |

### Long-open roles

- Share of boards with at least one posting open more than 180 days, percent: 60.7% (N=633)

### Rated usefulness, filtered vs unfiltered (placeholder)

Not run yet. For the first three design partners, two CSVs go out on the same day, one with reposts and stale roles included and one without, and each row is rated. The two precision figures are quoted only when both lists have at least 50 rated rows; until then nothing is reported here.

- Rated rows, filtered list: 0
- Rated rows, unfiltered list: 0

## Query or script

Script: `tell/backtest/noise.py`.

Repost share (bind :as_of = 2026-11-02T06:42:20.036241+00:00):

```sql
select b.vendor, p.role_family, count(*) as n, count(p.repost_of) as reposts
from postings p join ats_boards b on b.id = p.board_id
where p.first_seen_at > cast(:as_of as timestamptz) - interval '90 days'
  and p.first_seen_at <= cast(:as_of as timestamptz)
group by b.vendor, p.role_family
order by b.vendor, p.role_family;
```

Posting lifetime:

```sql
select p.role_family, count(*) as n,
       percentile_cont(0.5) within group (
         order by extract(epoch from p.closed_at - p.posted_at) / 86400.0) as median_days
from postings p
where p.closed_at is not null and p.posted_at is not null
  and p.closed_at <= cast(:as_of as timestamptz) and p.closed_at >= p.posted_at
group by rollup (p.role_family)
order by p.role_family nulls last;
```

Boards with a posting open more than 180 days:

```sql
select count(*) as boards,
       count(*) filter (where exists (
         select 1 from postings p
         where p.board_id = b.id
           and p.first_seen_at <= cast(:as_of as timestamptz)
           and (p.closed_at is null or p.closed_at > cast(:as_of as timestamptz))
           and coalesce(p.posted_at, p.first_seen_at)
               < cast(:as_of as timestamptz) - interval '180 days')) as long_open
from ats_boards b
where b.status = 'active';
```

## Caveats

- A repost is what tell's repost rule says it is (same fingerprint on the same board, closed within 30 days or same content within 7 days); a role re-listed after a longer gap counts as new.
- Lifetime is right-censored: postings still open are left out, so long-lived roles pull the median less than they should, and roles that closed before tell started watching are invisible.
- Lifetime needs the provider's posted_at; postings without one are left out of that section.
- Age for the 180-day rule is posted_at when the provider gives it, else the first time tell saw the posting, which understates the age of old roles on boards added late.

## Date

As of 2026-11-02; generated 2026-11-02T06:42:20+00:00.
