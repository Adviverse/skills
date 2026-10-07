# Adviverse agent skills

Skills are small, drop-in instruction packages (a folder with a `SKILL.md`) that teach an AI agent how to complete a common Adviverse workflow correctly, including the real gates (funding, moderation) that otherwise cause silent no-fills.

| Skill | Use it to |
|-------|-----------|
| `launch-campaign` | Create and launch an advertiser campaign end-to-end |
| `pull-reporting` | Pull spend, ROAS and reach reports |
| `onboard-publisher-and-embed` | Create a property + placement and get the embed tag |
| `build-audience` | Build a reusable audience from pixel events / rules |
| `manage-funds` | Check balance, deposit, request a payout |
| `moderate-creatives` | Admin: review and decide the moderation queue |

## Install

Drop a folder into `.claude/skills/` (Claude Code) or point your agent framework at this directory. Each skill leads with the `@adviverse/mcp` MCP server (`npx @adviverse/mcp`, env `ADVIVERSE_API_KEY`), with `@adviverse/sdk` and plain HTTP (`x-api-key`) fallbacks included.

## Validate

`node scripts/check-skills.mjs` fails if any tool listed in a skill is not in the MCP catalog.
