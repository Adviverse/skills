---
name: launch-campaign
description: Create and launch an advertiser campaign on Adviverse end-to-end.
---

# Launch a campaign

## When to use
The user wants to run ads: create a campaign (group + ad set), add a creative, fund the account and submit for review.

## MCP tools
Server: `npx @adviverse/mcp` with env `ADVIVERSE_API_KEY`.

- `create_campaign` — create the ad set (the real serving unit). `fields`: name, format, pricingModel, bidAmount, budgetDaily, targeting, etc.
- `create_creative` — create a creative (`type` image|native|popunder_url|push|html5|video, `landingUrl`, `assetUrl`, title/cta...). Omit `campaignId` for a library creative.
- `attach_creative` — attach a library creative to a campaign (`id`, `campaignId`, optional `weight`).
- `reach_estimate` — estimate audience size for a targeting spec before launching.
- `get_balance` — check available funds.
- `deposit_funds` — top up (`amount`, `provider`, `currency`).
- `submit_campaign` — submit the campaign for moderation review.
- `set_campaign_status` — pause or resume an approved campaign later. It does not check approval and is not part of launch.
- `get_campaign` — read status / review state.

## Steps
1. (Optional grouping) Create a campaign group: there is no MCP tool, use `POST /v1/advertiser/campaign-groups` `{name, objective}`. In the UI a "Campaign" is a group and an "Ad set" is the campaign you create next.
2. `create_campaign` with name, format, pricingModel, bidAmount (the DTO marks these optional, but the campaigns row needs them, so always send them), plus budgetDaily and targeting (geo, device, dayparting...). If you made a group in step 1, pass its id as `groupId` so the ad set is linked to it; otherwise it is created standalone, not under the group. Optionally run `reach_estimate` first.
3. `create_creative` with the landing URL and assets, then `attach_creative` to the campaign (or pass `campaignId` at creation).
4. `get_balance`; if low, `deposit_funds` so the campaign can deliver once approved. Instruction-based rails (wire, paxum) return instructions and credit only after confirmation.
5. `submit_campaign`. The campaign moves to `pending_review` and enters the moderation queue.
6. On approval the campaign becomes `active` automatically (rejection sets it to `rejected`). Poll `get_campaign` to see the outcome; do not call `set_campaign_status` to activate.

## Gotchas
- Gate: a creative serves only once it is moderation-approved and malware-clean. Submitting does not mean serving; wait for approval (it then activates automatically).
- Advice, not a server gate: fund the account so the campaign can deliver once approved. Submit does not check balance and will not reject a zero balance, but an unfunded campaign cannot spend.
- House/local placements auto-approve; marketplace traffic does not.
- Creative size must match the placement size or it will no-fill.
- Use `idempotencyKey` on `create_campaign` to make retries safe.

## SDK / HTTP fallback
```ts
import { AdNetClient } from '@adviverse/sdk';
const c = new AdNetClient({ baseUrl: 'https://adviverse.com', apiKey: process.env.ADVIVERSE_API_KEY });
// The `as never` casts below are because the body is loosely typed here; you can
// use the real request types (e.g. CreateCampaignRequest) instead.
const camp = await c.campaigns.create({
  name: 'Spring', groupId: GROUP_ID, // groupId links the ad set to the group from step 1
  format: 'banner', pricingModel: 'cpm', bidAmount: 2.5, budgetDaily: 50,
} as never);
const cr = await c.creatives.create({ type: 'image', landingUrl: 'https://example.com', assetUrl: '...' } as never);
await c.creatives.attach(cr.id, camp.id);
await c.campaigns.submit(camp.id);
```
```bash
curl -X POST https://adviverse.com/v1/advertiser/campaigns -H "x-api-key: $KEY" -H "content-type: application/json" -d '{"name":"Spring","groupId":"'$GROUP'","format":"banner","pricingModel":"cpm","bidAmount":2.5,"budgetDaily":50}'
curl -X POST https://adviverse.com/v1/advertiser/creatives/$CREATIVE/attach -H "x-api-key: $KEY" -H "content-type: application/json" -d '{"campaignId":"'$CAMP'"}'
curl -X POST https://adviverse.com/v1/advertiser/campaigns/$CAMP/submit -H "x-api-key: $KEY"
curl https://adviverse.com/v1/finance/balance -H "x-api-key: $KEY"
```
