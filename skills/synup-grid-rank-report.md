---
name: synup-grid-rank-report
description: Track local keyword rankings for a Synup location and generate a grid-rank heatmap report, then read the per-cell coordinates and ranks.
api: Synup API v4
base_url: https://api.synup.com/api/v4
generated: '2026-08-13'
method: generated
source: openapi/synup-api-openapi.yml
operations:
  - listKeywords
  - POST /locations/keywords
  - POST /locations/keywords/archive
  - GET /locations/{locationId}/keywords-performance
  - POST /locations/ranking-analytics-timeline
  - POST /locations/ranking-sitewise-histogram
  - createGridReport
  - getAllGridReports
  - getGridReport
---

# Produce a grid-rank heatmap

A single "we rank #3" number hides the geography. The grid endpoint returns rank per cell
plus coordinates, so you render the heatmap yourself instead of receiving a screenshot.

## Steps

1. **Confirm rankings are enabled.** `SY20009` (*Rankings not enabled for this account*) is
   the gate; there is no capability endpoint, so treat that code as the feature check.
2. **Manage keywords.** `listKeywords` — `GET /locations/{locationId}/keywords`. Add with
   `POST /locations/keywords`; keywords are 2–255 characters (`SY20201`, `SY20202`), must be
   unique per location (`SY20106`), and the account keyword limit is enforced (`SY20103`).
   Retire with `POST /locations/keywords/archive`.
3. **Read performance.** `GET /locations/{locationId}/keywords-performance` for the current
   position set, `POST /locations/ranking-analytics-timeline` for the rollup over time, and
   `POST /locations/ranking-sitewise-histogram` for the distribution.
4. **Create the grid report.** `createGridReport` — `POST /create-grid-report`. This is an
   **asynchronous job**, not a read: it returns a report handle.
5. **Collect it.** Poll `getAllGridReports` (`GET /locations/{locationId}/grid-reports`) or
   fetch a known one with `getGridReport` (`GET /grid-report/{reportId}`).

## Prefer the event over the poll

The webhook `rankings.gridrank_report_ready` fires when the report is complete, and
`rankings.sov_snapshot_completed` fires for share-of-voice snapshots. There is no documented
rate limit and no `Retry-After` header, so a tight poll loop has no published backoff signal
to obey — use the webhook where the account has one configured.
