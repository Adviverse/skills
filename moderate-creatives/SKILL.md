---
name: moderate-creatives
description: Admin: review the creative moderation queue and approve/reject.
---

# Moderate creatives

## When to use
An admin needs to clear the moderation queue so submitted creatives can serve.

## MCP tools
Server: `npx @adviverse/mcp` with env `ADVIVERSE_API_KEY` (admin-scoped key).

- `admin_moderation_queue` — list pending moderation items (optional `type`). The queue holds more than creatives: entity types are `creative`, `campaign`, `promote_campaign` and `property`.
- `admin_decide_moderation` — approve or reject a queue item (`id` is the QUEUE-ITEM id, not the entity id; `decision` approved|rejected, `notes`).
- `admin_rescan_creative` — re-run the malware scan on a creative (takes a CREATIVE id, i.e. the queue item's `entity_id`, not the queue-item id).
- `get_creative` — inspect creative details.

## Steps
1. `admin_moderation_queue` to list pending items.
2. Inspect each (landing URL, assets, `get_creative`); rescan with `admin_rescan_creative` if the scan result is missing or stale.
3. `admin_decide_moderation` with `decision: approved` or `rejected`, and `notes` giving the reason when rejecting.
4. Re-list the queue to confirm it is cleared.

## Gotchas
- Two different ids: decide uses the queue-item id (`/moderation/:id/decide`); rescan uses the creative id (`/moderation/:creativeId/rescan`). Do not mix them up.
- Approving a `campaign` item activates that campaign; rejecting it sets it to `rejected`.
- Requires an admin-scope key; other keys get 403.
- Approval alone is not enough to serve: the creative must also be malware-clean, active and size-matched.
- Always give a reason on rejection so the advertiser can fix and resubmit.
- Decisions are audited (`admin_audit_log`).

## SDK / HTTP fallback
```ts
const queue = await client.admin.moderation({ type: 'creative' });
await client.admin.decideModeration(queue[0].id /* queue-item id */, 'rejected', 'Landing page mismatch');
```
```bash
curl https://adviverse.com/v1/admin/moderation -H "x-api-key: $KEY"
curl -X POST https://adviverse.com/v1/admin/moderation/$ID/decide -H "x-api-key: $KEY" -H "content-type: application/json" -d '{"decision":"approved"}'
```
