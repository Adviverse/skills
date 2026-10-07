---
name: pull-reporting
description: Pull advertiser or publisher performance reports (spend, ROAS, reach).
---

# Pull reporting

## When to use
The user asks how campaigns or placements are performing: spend, revenue, ROAS, reach/frequency, geo/device breakdowns.

## MCP tools
Server: `npx @adviverse/mcp` with env `ADVIVERSE_API_KEY`.

- `advertiser_report` — advertiser spend/performance.
- `publisher_report` — publisher revenue/delivery.
- `report_roas` — daily spend, conversions, conversion value, ROAS.
- `report_reach` — unique reach, average frequency, frequency histogram.
- `report_breakdown` — breakdown by a dimension.
- `report_geo` — performance by geography.
- `report_realtime` — near-real-time delivery.
- `publisher_placements_report` — per-placement publisher report.

## Steps
1. Pick the report: advertiser (`advertiser_report`), publisher (`publisher_report`), ROAS (`report_roas`) or reach (`report_reach`).
2. Set the date range with `from` and `to` (ISO date/datetime, inclusive). Optionally `dim` on advertiser/publisher reports; accepted values are `day` (default), `hour`, `geo`, `device`, `format`, `placement_id`, `campaign_id`. `report_breakdown` uses a different set (`bdim`): os, browser, deal, page, page_url, app, property, region, make, model, sub1..sub5.
3. Read the returned rows; summarize totals and call out the top and bottom performers.
4. For a drill-down, call `report_breakdown` or `report_geo` with the same range.

## Gotchas
- Always pass an explicit `from`/`to`; defaults may be a short window.
- ROAS needs conversions tracked (pixel or `track_conversion`); empty ROAS usually means no conversion events.
- Reach is unique users, so it is not additive across days.
- Role matters: advertiser reports need an advertiser key, publisher reports a publisher key.

## SDK / HTTP fallback
```ts
const rows = await client.reporting.advertiser({ from: '2026-09-01', to: '2026-09-30', dim: 'day' });
const roas = await client.reporting.roas({ from: '2026-09-01', to: '2026-09-30' });
```
```bash
curl "https://adviverse.com/v1/reporting/advertiser?from=2026-09-01&to=2026-09-30&dim=day" -H "x-api-key: $KEY"
curl "https://adviverse.com/v1/reporting/roas?from=2026-09-01&to=2026-09-30" -H "x-api-key: $KEY"
curl "https://adviverse.com/v1/reporting/reach?from=2026-09-01&to=2026-09-30" -H "x-api-key: $KEY"
curl "https://adviverse.com/v1/reporting/publisher?from=2026-09-01&to=2026-09-30" -H "x-api-key: $KEY"
```
