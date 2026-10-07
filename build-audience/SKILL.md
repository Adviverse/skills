---
name: build-audience
description: Build a reusable audience from pixel events / rules.
---

# Build an audience

## When to use
The user wants to retarget or target visitors based on events (viewed product, abandoned cart, purchased) captured by the Adviverse pixel.

## MCP tools
Server: `npx @adviverse/mcp` with env `ADVIVERSE_API_KEY`.

- `list_pixels` — list the account's pixels.
- `create_pixel` — create a pixel (returns id + base snippet).
- `pixel_events` — inspect events the pixel has received.
- `audience_suggestions` — ready-made audiences suggested from harvested events.
- `estimate_audience` — estimate reach for a rule without creating it.
- `create_audience` — create an audience (name, type `rule`, `rule`, optional `pixelId`).
- `create_rule_audience` — create a rule-based audience.
- `list_audiences` — list audiences.
- `update_audience` — change name/rule; membership re-resolves.
- `update_campaign` — set the audience on a campaign's targeting.

## Steps
1. `list_pixels`; if none, `create_pixel` and install the snippet on the site.
2. Confirm data is flowing with `pixel_events`.
3. Try `audience_suggestions` for a quick start, or define a rule (flat rule or RuleNode tree) on events.
4. `estimate_audience` with the rule (and `pixelId`) to check size.
5. `create_audience` (or `create_rule_audience`) with the rule.
6. Reference the audience in an ad set's targeting via `update_campaign`.

## Gotchas
- With multiple pixels (agencies), scope with `pixelId`; membership is scoped per pixel.
- Tiny audiences may not serve; check the estimate first.
- Consent mode matters: contextual/none traffic is not added to personalized audiences.
- Rules re-resolve membership on update, so changes are not instant.

## SDK / HTTP fallback
```ts
const est = await client.audiences.estimate(rule, pixelId);
const { id } = await client.audiences.create({ name: 'Cart abandoners', type: 'rule', rule, pixelId });
```
```bash
curl https://adviverse.com/v1/advertiser/audiences/suggestions -H "x-api-key: $KEY"
curl -X POST https://adviverse.com/v1/advertiser/audiences/estimate -H "x-api-key: $KEY" -H "content-type: application/json" -d '{"rule":{...}}'
curl -X POST https://adviverse.com/v1/advertiser/audiences -H "x-api-key: $KEY" -H "content-type: application/json" -d '{"name":"Cart abandoners","type":"rule","rule":{...}}'
```
