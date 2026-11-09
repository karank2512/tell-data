---
report: noise
title: "Noise handling: reposts, lifetimes and long-open roles"
as_of: 2026-11-09
generated_at: 2026-11-09T06:43:20+00:00
n: 14923
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

Cohort N: 14923.

## Numbers

### Reposts

- Share of postings that are reposts (first seen in the last 90 days), percent: 2.0% (N=14,923)

| vendor | postings | reposts | repost share |
| --- | --- | --- | --- |
| ashby | 8,508 | 187 | 2.2% (N=8,508) |
| greenhouse | 5,656 | 97 | 1.7% (N=5,656) |
| lever | 665 | 3 | 0.5% (N=665) |
| smartrecruiters | 94 | 4 | 4.3% (N=94) |

### Reposts by role family

| family | postings | reposts | repost share |
| --- | --- | --- | --- |
| customer | 1,365 | 12 | 0.9% (N=1,365) |
| data | 179 | 9 | 5.0% (N=179) |
| engineering | 3,063 | 53 | 1.7% (N=3,063) |
| exec | 570 | 7 | 1.2% (N=570) |
| finance_ops | 1,023 | 31 | 3.0% (N=1,023) |
| gtm_ops | 848 | 28 | 3.3% (N=848) |
| marketing | 737 | 16 | 2.2% (N=737) |
| ml_ai | 367 | 12 | 3.3% (N=367) |
| other | 3,576 | 46 | 1.3% (N=3,576) |
| product_design | 995 | 29 | 2.9% (N=995) |
| sales | 2,200 | 48 | 2.2% (N=2,200) |

### Posting lifetime

- Median posting lifetime, closed_at - posted_at, days (all families): 53.1 days (N=3,281)

| family | median lifetime |
| --- | --- |
| customer | 54.7 days (N=328) |
| data | 37.9 days (N=45) |
| engineering | 70.8 days (N=567) |
| exec | 61.0 days (N=126) |
| finance_ops | 49.3 days (N=279) |
| gtm_ops | 43.9 days (N=215) |
| marketing | 44.8 days (N=193) |
| ml_ai | 73.2 days (N=69) |
| other | 45.2 days (N=774) |
| product_design | 60.9 days (N=232) |
| sales | 64.8 days (N=453) |

### Long-open roles

- Share of boards with at least one posting open more than 180 days, percent: 60.8% (N=636)

### Rated usefulness, filtered vs unfiltered (placeholder)

Not run yet. For the first three design partners, two CSVs go out on the same day, one with reposts and stale roles included and one without, and each row is rated. The two precision figures are quoted only when both lists have at least 50 rated rows; until then nothing is reported here.

- Rated rows, filtered list: 0
- Rated rows, unfiltered list: 0

## Query or script

Script: `tell/backtest/noise.py`.

Repost share (bind :as_of = 2026-11-09T06:43:20.981552+00:00):

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

As of 2026-11-09; generated 2026-11-09T06:43:20+00:00.
