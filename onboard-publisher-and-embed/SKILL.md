---
name: onboard-publisher-and-embed
description: Onboard a publisher: create a property + placement and get the embed tag.
---

# Onboard a publisher and embed the tag

## When to use
The user has a site or app and wants to monetize it: register it, define an ad slot, and get the snippet to paste.

## MCP tools
Server: `npx @adviverse/mcp` with env `ADVIVERSE_API_KEY`.

- `create_property` — register a property. `fields`: kind (website|cli|mobile_app|ctv), domain, name.
- `verify_property` — trigger ownership verification.
- `create_placement` — create an ad slot. `fields`: propertyId, name, format, floorCpm.
- `get_placement_tag` — fetch the embed tag for a placement.
- `list_placements` — list existing placements.
- `get_app_ads_txt` — get the app-ads.txt content for app inventory.

## Steps
1. `create_property` with the domain (kind `website`).
2. `verify_property` and complete the returned verification step (DNS/file) if required.
3. `create_placement` with `propertyId`, a name, `format` and size, and an optional `floorCpm`.
4. `get_placement_tag` with the placement id (`GET /v1/publisher/placements/:id/tag`).
5. Give the user the returned `<script ... data-tag>` snippet to paste where the ad should render (before `</body>` or in the slot container).

## Gotchas
- Ads only fill when an approved, funded, size-matched campaign exists; test with the test tag before judging fill.
- Popunder: the SDK arms at most one popunder per page.
- Unverified properties may be restricted from serving.
- Consent config is per property, not per placement.

## SDK / HTTP fallback
```ts
// `as never` casts: bodies are loosely typed here; use the real request types if you prefer.
const prop = await client.properties.create({ kind: 'website', domain: 'example.com' });
const pl = await client.placements.create({ propertyId: prop.id, name: 'Sidebar', format: 'banner', floorCpm: 0.5 } as never);
const { tag } = await client.placements.tag(pl.id);
```
```bash
curl -X POST https://adviverse.com/v1/publisher/properties -H "x-api-key: $KEY" -H "content-type: application/json" -d '{"kind":"website","domain":"example.com"}'
curl -X POST https://adviverse.com/v1/publisher/properties/$PROP/verify -H "x-api-key: $KEY"
curl https://adviverse.com/v1/publisher/placements/$PLACEMENT/tag -H "x-api-key: $KEY"
```
