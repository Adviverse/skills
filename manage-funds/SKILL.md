---
name: manage-funds
description: Check balance and fund an advertiser account / request a payout.
---

# Manage funds

## When to use
The user wants to see their balance, top up an advertiser account, review deposits/payouts, or (publisher) cash out.

## MCP tools
Server: `npx @adviverse/mcp` with env `ADVIVERSE_API_KEY`.

- `get_balance` — available and held balance.
- `deposit_funds` — request a deposit (`amount`, `provider`, `currency`).
- `list_deposits` — deposit history.
- `request_payout` — publisher payout (`amount`, `method` crypto|paxum|wire, `destination`).
- `list_payouts` — payout history.
- `list_ledger` — immutable ledger entries.
- `get_statement` — account statement.

## Steps
1. `get_balance` to see available and held funds.
2. To fund: `deposit_funds` with amount and provider. Card/crypto return a checkout; wire and paxum return instructions (title, reference, lines) to send funds.
3. Confirm with `list_deposits`; instruction-based rails are credited only after the receipt is confirmed.
4. Publisher payout: `request_payout` with method and destination (`destination.email` for paxum, bank fields for wire).
5. Track with `list_payouts`; wire payouts settle once ops confirms they were sent.

## Gotchas
- The ledger is append-only: balances move only through ledger entries, never edited.
- Only configured payment rails work; unconfigured ones are refused, not mocked.
- Include the reference from wire/paxum instructions or the deposit cannot be matched.
- Tenant-network accounts transact in the tenant's platform token (the currency may not be ISO).

## SDK / HTTP fallback
```ts
// `as never` casts: bodies are loosely typed here; use the real request types if you prefer.
const bal = await client.finance.balance();
const dep = await client.finance.deposit({ amount: 500, provider: 'wire' } as never);
await client.finance.requestPayout({ amount: 100, method: 'crypto' } as never);
```
```bash
curl https://adviverse.com/v1/finance/balance -H "x-api-key: $KEY"
curl -X POST https://adviverse.com/v1/finance/deposit -H "x-api-key: $KEY" -H "content-type: application/json" -d '{"amount":500,"provider":"wire"}'
curl -X POST https://adviverse.com/v1/finance/payout -H "x-api-key: $KEY" -H "content-type: application/json" -d '{"amount":100,"method":"crypto"}'
```
