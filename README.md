# tell-data

Generated data for [tell](https://github.com/karank2512/tell-ai), the open-source
buying-signals pipeline. **Everything in this repository is written by `tell publish`
from the ingest workflow; do not edit it by hand.** Each commit is one ingest run
(four a day), and every published page links to the code commit that produced it.

Sources: the data comes from public job boards (Greenhouse, Lever, Ashby,
SmartRecruiters), SEC EDGAR and news headlines; see the code repository's
`docs/sources/` for each source's endpoint and legal note. No file holds a person's
name, email address or phone number.

## Layout

```
events/YYYY/MM/DD.jsonl
    One JSON object per line, one line per event observed that UTC day: event_id,
    company_id, source, type, occurred_at, observed_at, magnitude, url, payload,
    raw_ref, run_id. Append-only: lines are never rewritten, reordered or removed.
    This is the system of record, and the archive for events older than 180 days
    once they are pruned from the database.

postings/snapshots/YYYY-MM-DD/{vendor}-{identifier}.json
    The trimmed state of one job board that day: its open postings (external_id,
    title, role_family, location, posted_at, url). No descriptions. Kept 90 days.

site/
    The GitHub Pages root.
    index.html                      this week's top movers, methods, repo link
    companies/{domain}.html         account page: score, contributions, receipts
    weekly/{ISO-week}.html          the weekly edition
    health.html                     run-health per source
    api/v0/companies/{domain}.json  the per-domain JSON contract
    api/v0/weekly/latest.json       the current weekly edition
    api/v0/watchlists/{slug}.json   public watchlists only
    api/v0/health.json              run-health as JSON
```

The JSON contract, its schema and a Clay recipe are documented in the code repository
under `docs/api/`; the layout, retention and size budget under
`docs/methods/publishing.md`.

## Pages

GitHub Pages serves `site/` (branch `main`, folder `/site`). Account pages exist for
companies with at least one signal in the last 180 days; the rest are pruned to stay
inside the Pages size budget.
