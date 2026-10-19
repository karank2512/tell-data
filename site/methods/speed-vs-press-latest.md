---
report: speed-vs-press
title: "Speed vs press: Form D filings against the first funding headline"
as_of: 2026-10-19
generated_at: 2026-10-19T07:02:24+00:00
n: 334
quotable: true
flagged_numbers: 9
script: tell/backtest/speed_vs_press.py
private: false
---

# Speed vs press: Form D filings against the first funding headline

Dated report written by `tell backtest speed-vs-press` (spec section 13.1). The SQL below is the implementation: it runs as printed, with `:as_of` bound to the evaluation instant. Numbers are quoted from this file only, with their N; the method asks for at least 100 filings.

## Question

How long after a Form D filing does the first news mention appear, and how fast would tell have alerted?

## Cohort

Every Form D (isAmendment = false) filed in the 90 days before the evaluation instant by an issuer linked to a monitored company by a trusted link: rules R0-R4 with no guard failure recorded, a reviewer's link, or the issuer's CIK when that CIK was set by such a link or has a trusted source. Review (R5), unlinked (R6) and guard-failed matches are excluded. Split in two and reported pooled: monitored before the filing (a YC directory company, or one that existed before the filing date) and found from the filing (created on or after the filing date by issuer-first discovery, whose alert time is the link time). Press: news_funding events of that company dated from 14 days before to 60 days after the filing date (collected first with Google News when:120d by `tell ingest --source news --cohort formd --when-days 120`); for a one-word brand not from the YC directory the headline must carry the full legal name.

Cohort N: 334.

## Numbers

### Press after filing

- Cohort N: Form D filings (isAmendment = false) linked to a monitored company: 334
- Share with any press within the window (filing - 14d to filing + 60d): 16.2% (N=334)
- Filings with any press within the window (the N behind the lag numbers): 54
- Share with no press within 60 days: 83.8% (N=334)
- Median of first_press_at - filing_date, days: 4.5 days (N=54)
- p75 of first_press_at - filing_date, days: 20.2 days (N=54)

### Who came first

- Filing first, among filings with press (press strictly after the filing date): 63.0% (N=54)
- Filing first, over all filings (no press counts as filing first): 94.0% (N=334)

### How fast tell alerts

- Median alert_at - filing_date, hours: 895.3 h (N=334)
- Median alert_at - filing_date, hours, observed filings only: 26.6 h (N=3; too small to quote)
- Filings whose alert_at is observed (a scheduled run within 36 h of the filing date): 3
- Filings whose alert_at is estimated (filing date + 1 day 02:00 UTC + the 6-hour cadence, 32 h after the filing date): 18

### By cohort

Monitored before the filing: a YC directory company, or one tell had before the filing date. Found from the filing: created on or after the filing date by issuer-first discovery; its alert time is the link time. Pooled: both.

| Number | Monitored before the filing | Found from the filing | Pooled |
| --- | --- | --- | --- |
| Filings | 21 | 313 | 334 |
| Share with any press | 33.3% (N=21; too small to quote) | 15.0% (N=313) | 16.2% (N=334) |
| Share with no press within 60 days | 66.7% (N=21; too small to quote) | 85.0% (N=313) | 83.8% (N=334) |
| Median lag, days | 12.0 days (N=7; too small to quote) | 4.0 days (N=47) | 4.5 days (N=54) |
| p75 lag, days | 24.5 days (N=7; too small to quote) | 19.5 days (N=47) | 20.2 days (N=54) |
| Filing first, among filings with press | 71.4% (N=7; too small to quote) | 61.7% (N=47) | 63.0% (N=54) |
| Filing first, over all filings | 90.5% (N=21; too small to quote) | 94.2% (N=313) | 94.0% (N=334) |
| Median alert_at - filing_date, hours | 32.0 h (N=21; too small to quote) | 991.3 h (N=313) | 895.3 h (N=334) |
| Alert times observed | 3 | 0 | 3 |
| Alert times estimated | 18 | 0 | 18 |
| Alert times at the link | 0 | 313 | 313 |
| Press-ambiguous filings (one-word brand) | 14 (N=21; too small to quote) | 147 (N=313) | 161 (N=334) |

## Query or script

Script: `tell/backtest/speed_vs_press.py`.

Implementation (bind :as_of = 2026-10-19T07:02:24.848156+00:00, :cadence_hours = 6, :observed_window_hours = 36):

```sql
with links as (
  select e.event_id, e.company_id, e.payload->>'cik' as cik,
         e.payload->'match'->>'rule' as rule,
         case jsonb_typeof(e.payload->'match'->'guard_failed')
           when 'boolean' then (e.payload->'match'->>'guard_failed')::bool
           when 'array' then jsonb_array_length(e.payload->'match'->'guard_failed') > 0
           when 'object' then e.payload->'match'->'guard_failed' <> '{}'::jsonb
           when 'string' then e.payload->'match'->>'guard_failed' <> ''
           when 'number' then (e.payload->'match'->>'guard_failed')::numeric <> 0
           else false end as guard_failed
  from events e
  where e.source = 'edgar' and e.company_id is not null and e.payload->'match' is not null),
trusted as (
  select event_id, company_id, cik from links
  where rule = 'manual' or (rule in ('R0', 'R1', 'R2', 'R3', 'R4') and not guard_failed)),
d as (
  select e.event_id, e.company_id, e.observed_at,
         (e.occurred_at at time zone 'UTC')::date as filed_on,
         coalesce((e.payload->>'backfill')::bool, false) as backfill,
         case when coalesce(e.payload->'match'->>'resolved_at', '') ~ '^\d{4}-'
              then (e.payload->'match'->>'resolved_at')::timestamptz end as linked_at,
         c.source, c.created_at
  from events e join companies c on c.id = e.company_id
  where e.type = 'form_d_filed' and (e.payload->>'is_amendment')::bool = false
    and e.occurred_at >= cast(:as_of as timestamptz) - interval '90 days'
    and e.occurred_at < cast(:as_of as timestamptz)
    and e.company_id is not null
    and (e.event_id in (select event_id from trusted)
         or (e.payload->'match' is null and c.cik = e.payload->>'cik'
             and (c.cik_source in ('issuer_first_a', 'issuer_first_b', 'r1', 'guarded_r2', 'manual')
                  or exists (select 1 from trusted t
                             where t.company_id = c.id and t.cik = c.cik))))),
names as (
  select c.id as company_id, x.kind,
         trim(regexp_replace(lower(x.value), '\W+', ' ', 'g')) as norm
  from companies c
  cross join lateral (values ('name', c.name), ('legal', c.legal_name)) as x(kind, value)
  where c.id in (select company_id from d) and x.value is not null),
keys as (
  select company_id, kind,
         regexp_replace(
           regexp_replace(norm,
             '^(.*\S) (inc|incorporated|llc|ltd|limited|corp|corporation|co|plc|gmbh|pbc)$',
             '\1'),
           '^(.*\S) (inc|incorporated|llc|ltd|limited|corp|corporation|co|plc|gmbh|pbc)$',
           '\1') as brand
  from names),
amb as (
  select b.company_id, l.brand as full_brand
  from keys b left join keys l on l.company_id = b.company_id and l.kind = 'legal'
  where b.kind = 'name' and b.brand <> '' and position(' ' in b.brand) = 0),
f as (
  select d.*, d.filed_on::timestamp at time zone 'UTC' as filed_at,
         case when d.source <> 'yc' and d.created_at >= d.filed_on::timestamp at time zone 'UTC'
              then 'found' else 'monitored' end as cohort,
         amb.company_id is not null as press_ambiguous,
         case when amb.company_id is not null and d.source <> 'yc'
                   and position(' ' in coalesce(amb.full_brand, '')) > 0
              then amb.full_brand end as full_brand
  from d left join amb on amb.company_id = d.company_id),
o as (
  select f.*,
         (not f.backfill and f.observed_at >= f.filed_at
          and f.observed_at <= f.filed_at
                               + cast(:observed_window_hours as integer) * interval '1 hour')
           as prompt
  from f),
a as (
  select o.*,
         o.cohort = 'monitored' and o.prompt as alert_observed,
         case when o.cohort = 'found'
              then greatest(case when o.prompt then o.observed_at
                                 else o.filed_at + interval '1 day 2 hours'
                                      + cast(:cadence_hours as integer) * interval '1 hour' end,
                            coalesce(o.linked_at, o.created_at))
              when o.prompt then o.observed_at
              else o.filed_at + interval '1 day 2 hours'
                   + cast(:cadence_hours as integer) * interval '1 hour' end as alert_at
  from o),
p as (
  select a.event_id, min((n.occurred_at at time zone 'UTC')::date) as first_press_on
  from a join events n on n.company_id = a.company_id
  where n.type = 'news_funding'
    and (n.occurred_at at time zone 'UTC')::date between a.filed_on - 14 and a.filed_on + 60
    and (a.full_brand is null
         or position(' ' || a.full_brand || ' ' in
                     ' ' || trim(regexp_replace(lower(coalesce(n.payload->>'title', '')),
                                                '\W+', ' ', 'g')) || ' ') > 0)
  group by a.event_id)
select coalesce(cohort, 'pooled') as cohort,
       count(*) as n, count(p.event_id) as with_press,
       percentile_cont(0.5) within group (order by first_press_on - filed_on) as median_lag_days,
       percentile_cont(0.75) within group (order by first_press_on - filed_on) as p75_lag_days,
       sum(case when first_press_on > filed_on then 1 else 0 end) as filing_first,
       percentile_cont(0.5) within group (
         order by extract(epoch from alert_at - filed_at) / 3600.0
       ) as median_alert_hours,
       percentile_cont(0.5) within group (
         order by extract(epoch from alert_at - filed_at) / 3600.0
       ) filter (where alert_observed) as median_observed_alert_hours,
       sum(case when alert_observed then 1 else 0 end) as alert_observed,
       sum(case when cohort = 'monitored' and not alert_observed then 1 else 0 end)
         as alert_estimated,
       sum(case when cohort = 'found' then 1 else 0 end) as alert_linked,
       sum(case when press_ambiguous then 1 else 0 end) as press_ambiguous,
       sum(case when full_brand is not null then 1 else 0 end) as full_brand_required
from a left join p using (event_id)
group by grouping sets ((cohort), ());
```

## Caveats

- Google News coverage is incomplete for small companies; the no-press share is reported, not hidden.
- first_press_at is the earliest funding headline the news collector kept (name in the headline, funding regex match); a story that never names the company in its headline is invisible.
- The press window (from 14 days before to 60 days after the filing date, as SPEC 13.1 sets it) misses same-round press published more than 14 days before the filing: such a round counts as no press, or as filing first when later press exists, so the filing-first shares lean high.
- EDGAR publishes a day's form index at about 02:02 UTC the next day, so no scheduled run can see a filing before filing date + 26 h. alert_at is the event's observed_at only when a scheduled run (not the quarterly backfill) saw the filing within 36 h of the filing date; filings from the quarterly data sets, from a catch-up run days later, or with observed_at before the filing date get the estimate filing date + 1 day 02:00 UTC + the 6-hour cadence, 32 h after the filing date. Observed: 3; estimated: 18. The estimate adds one full 6-hour cadence after the index appears, the longest wait for the next run. With one run a day (TELL_RUNS_PER_DAY not 4) that wait is up to 24 h; pass the 24-hour cadence for such a period.
- Links: the 2026-10-14 production audit found 20 of 41 automatic Form D links were the wrong company, so only links that are trusted after the guards are counted; a wrong link that passed every guard is still in the cohort.
- Found-from-the-filing companies (issuer-first discovery, ADR 0035) are monitored because they filed: their alert time includes how long discovery took to link the filing, and they have no hiring history before it. Report them apart; the pooled row mixes two different questions.
- Press-ambiguous brands: tell has no dictionary to tell a common word from a coined one, so every one-word brand is flagged (161 of 334 filings). Only for companies not from the YC directory with a longer legal name does the headline have to carry that name (4 filings); a YC one-word brand still counts any headline naming it.
- Changes from the spec's SQL, all needed to run it as a dated report: `now()` is the bound `:as_of` instant, so a dated report can be regenerated. Filings are keyed by `event_id`, not `company_id`: the spec's `group by n.company_id` merges two filings of one company into one press date. Dates are taken in UTC (`at time zone 'UTC'`), independent of the session time zone. The cohort keeps only trusted links, as step 1 asks for linked issuers: rules R1-R4 with no `payload.match.guard_failed`, a reviewer's link (`rule = 'manual'`), or a CIK match whose CIK was set by such a link; R5, R6 and guard failures are out. `alert_at` is `observed_at` only for a scheduled observation within `:observed_window_hours` of the filing date; otherwise it is the filing date + 1 day 02:00 UTC (the daily index) + `:cadence_hours`. Added columns: p75 lag, the median alert lag (`alert_at` per step 3) over all filings and over observed ones, and the observed and estimated counts. The news type name `news_funding` is unchanged. Rule R0 (an EDGAR event linked by a CIK with a trusted `companies.cik_source`, ADR 0035) is a trusted rule, and a CIK match with no match record is in when the company's `cik_source` is set. The cohort is split, every row grouped by `grouping sets ((cohort), ())`: 'monitored' (the company is from the YC directory, `source = 'yc'`, or existed before the filing date) and 'found' (created on or after the filing date and not from the YC directory: issuer-first discovery found it from this filing). The pooled row is both. For a 'found' filing `alert_at` is the link time (`payload.match.resolved_at`, else `companies.created_at`) when that is later than the scheduled alert, so the time discovery took counts. Press-ambiguous companies: a brand of one word (after dropping a trailing legal form, PBC included) is flagged. For a company not from the YC directory whose legal name is longer, a funding headline counts only when it contains that full name.

## Date

As of 2026-10-19; generated 2026-10-19T07:02:24+00:00.
