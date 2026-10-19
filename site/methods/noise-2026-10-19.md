---
report: noise
title: "Noise handling: reposts, lifetimes and long-open roles"
as_of: 2026-10-19
generated_at: 2026-10-19T07:02:24+00:00
n: 10588
quotable: false
flagged_numbers: 4
script: tell/backtest/noise.py
private: false
---

# Noise handling: reposts, lifetimes and long-open roles

Dated report written by `tell backtest noise` (spec section 13.3): how much of what the boards show is reposts and long-open roles, and how long postings live.

**Not quotable:** 1 headline number has N under 30.

## Question

How much of what job boards show is noise (reposted and long-open roles), and how long does a posting live?

## Cohort

Repost share: postings first seen in the 90 days before the evaluation instant. Lifetime: postings closed by then that carry the provider's posted_at. Long-open share: boards with status active.

Cohort N: 10588.

## Numbers

### Reposts

- Share of postings that are reposts (first seen in the last 90 days), percent: 0.1% (N=10,588)

| vendor | postings | reposts | repost share |
| --- | --- | --- | --- |
| ashby | 5,651 | 4 | 0.1% (N=5,651) |
| greenhouse | 4,294 | 1 | 0.0% (N=4,294) |
| lever | 558 | 0 | 0.0% (N=558) |
| smartrecruiters | 85 | 1 | 1.2% (N=85) |

### Reposts by role family

| family | postings | reposts | repost share |
| --- | --- | --- | --- |
| customer | 916 | 0 | 0.0% (N=916) |
| data | 123 | 0 | 0.0% (N=123) |
| engineering | 2,263 | 1 | 0.0% (N=2,263) |
| exec | 379 | 0 | 0.0% (N=379) |
| finance_ops | 645 | 1 | 0.2% (N=645) |
| gtm_ops | 592 | 0 | 0.0% (N=592) |
| marketing | 484 | 0 | 0.0% (N=484) |
| ml_ai | 266 | 0 | 0.0% (N=266) |
| other | 2,551 | 1 | 0.0% (N=2,551) |
| product_design | 730 | 1 | 0.1% (N=730) |
| sales | 1,639 | 2 | 0.1% (N=1,639) |

### Posting lifetime

- Median posting lifetime, closed_at - posted_at, days (all families): 45.4 days (N=3; too small to quote)

| family | median lifetime |
| --- | --- |
| customer | 53.5 days (N=1; too small to quote) |
| other | 45.4 days (N=1; too small to quote) |
| sales | 28.4 days (N=1; too small to quote) |

### Long-open roles

- Share of boards with at least one posting open more than 180 days, percent: 63.1% (N=575)

### Rated usefulness, filtered vs unfiltered (placeholder)

Not run yet. For the first three design partners, two CSVs go out on the same day, one with reposts and stale roles included and one without, and each row is rated. The two precision figures are quoted only when both lists have at least 50 rated rows; until then nothing is reported here.

- Rated rows, filtered list: 0
- Rated rows, unfiltered list: 0

## Query or script

Script: `tell/backtest/noise.py`.

Repost share (bind :as_of = 2026-10-19T07:02:24.848156+00:00):

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

As of 2026-10-19; generated 2026-10-19T07:02:24+00:00.
