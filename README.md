# GotCashback MCP server

**Endpoint:** `https://mcp.gotcashback.com` (streamable HTTP) · **Docs:** https://www.gotcashback.com/mcp-server/ · **Registry name:** `com.gotcashback/gotcashback`

GotCashback compares the cashback rates that multiple cashback portals currently pay for the same
online or in-store retailer, and finds discounted gift cards from multiple sellers, across 30+
countries. Rates are refreshed several times a day. Every result carries the tracked GotCashback
link for the portal; cashback is only credited when the purchase starts from that link.

All comparison tools work **without an account**. Signing in with a GotCashback account (OAuth 2.1,
dynamic client registration, PKCE) unlocks the personal tools: profile, favorite stores, and
cashback / gift card alerts.

This repository holds the public metadata for the hosted server (`server.json` for the
[Official MCP Registry](https://registry.modelcontextprotocol.io)). The server itself is
closed-source and operated by GotCashback.

## Connect

**Claude.ai / Claude Desktop** — Settings → Connectors → *Add custom connector* → URL
`https://mcp.gotcashback.com`. Sign in when you first use a personal tool.

**Claude Code**
```bash
claude mcp add --transport http gotcashback https://mcp.gotcashback.com
```

**Cursor** (`.cursor/mcp.json`) / **VS Code** (`.vscode/mcp.json`)
```json
{
  "mcpServers": {
    "gotcashback": { "url": "https://mcp.gotcashback.com" }
  }
}
```
(VS Code uses the key `servers` instead of `mcpServers`.)

**ChatGPT** — Settings → Connectors → *Create* → MCP server URL `https://mcp.gotcashback.com`, authentication OAuth.

## Tools

| Tool | Auth | Kind | What it does |
|---|---|---|---|
| `get_cashback_rates_by_store_name` | none | read | Current cashback rates from every portal for a store, by name and country |
| `get_cashback_rates_by_store_id` | none | read | Same, by GotCashback store ID |
| `get_stores_by_name` | none | read | Search stores by name |
| `get_stores_by_country` | none | read | List stores available in a country |
| `get_countries` | none | read | Supported countries |
| `get_portals` | none | read | List cashback portals and payout terms |
| `get_portals_by_name` | none | read | Search portals by name |
| `get_portal_by_id` | none | read | Portal details by ID |
| `get_best_deals_by_brand` | none | read | Best current deals for a brand |
| `get_best_deals_by_category` | none | read | Best current deals in a category |
| `get_gift_cards_by_store_name` | none | read | Discounted gift cards for a store, by name |
| `get_gift_cards_by_store_id` | none | read | Same, by store ID |
| `get_my_profile` | OAuth `profile` | read | The signed-in user's profile |
| `get_my_favorite_stores` | OAuth `favorites` | read | The user's favorite stores |
| `toggle_favorite_store` | OAuth `favorites` | write | Add or remove a favorite store |
| `get_my_alerts` | OAuth `alerts` | read | The user's cashback and gift card alerts |
| `set_store_alert` | OAuth `alerts` | write | Create or update an alert for a store |
| `remove_store_alert` | OAuth `alerts` | write (destructive) | Delete an alert |

Every tool declares `title`, `readOnlyHint` and `destructiveHint`.

## Authentication

- Protected resource metadata: https://mcp.gotcashback.com/.well-known/oauth-protected-resource
- Authorization server: https://www.gotcashback.com/ (metadata at `/.well-known/oauth-authorization-server`)
- Dynamic client registration: `POST https://www.gotcashback.com/connect/register` — public clients
  (`token_endpoint_auth_method: none`) and confidential clients (`client_secret_post` / `client_secret_basic`);
  PKCE S256 required. Scopes: `profile`, `favorites`, `alerts`, `offline_access`.
- Details for agents: https://www.gotcashback.com/auth.md

## Links

- Server card: https://mcp.gotcashback.com/.well-known/mcp/server-card.json
- Agent documentation index: https://www.gotcashback.com/llms.txt
- Privacy policy: https://www.gotcashback.com/Privacy/ · Terms: https://www.gotcashback.com/Terms/
- Support: support@gotcashback.com
