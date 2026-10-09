# GotCashback MCP server

**Endpoint:** `https://mcp.gotcashback.com` (Streamable HTTP) · **Docs:** https://www.gotcashback.com/mcp-server/ · **Registry name:** `com.gotcashback/gotcashback` · **Version:** 1.1.1

GotCashback is a cashback comparison service, not a cashback portal. It compares the current cashback
rates published by **113 cashback portals across 42 countries (40 in the United States)** for online
and in-store retailers, and finds discounted gift cards from multiple sellers. Rates are refreshed
several times a day. Founded in 2020; operated by MNP Enterprises, LLC.

The coverage numbers above are as of October 2026; the live figures are at
https://api.gotcashback.com/v1/coverage and in the server card.

All comparison tools work **without an account**. Signing in with a GotCashback account (OAuth 2.1,
dynamic client registration, PKCE) unlocks the account tools: profile, favorite stores, and cashback /
gift card alerts.

Every store, cashback rate, gift card and portal sign-up bonus in a result carries a tracked
GotCashback link (`url`, plus `gift_cards_url` where relevant). Cashback is only credited, gift card
discounts only apply, and sign-up bonuses are only granted when the purchase starts from that link.

This repository holds the public metadata for the hosted server: `server.json` for the
[Official MCP Registry](https://registry.modelcontextprotocol.io) and this documentation. The server
itself is closed-source and operated by GotCashback. See [CHANGELOG.md](CHANGELOG.md) for what changed.

## Connect

**Claude.ai / Claude Desktop** — Settings → Connectors → *Add custom connector* → URL
`https://mcp.gotcashback.com`. Enable it in a conversation; Claude calls the tools automatically. You
are asked to sign in the first time you use an account tool.

**Claude Code**

```bash
claude mcp add --transport http gotcashback https://mcp.gotcashback.com
```

Run `/mcp` inside Claude Code to check the connection and to sign in for the account tools.

**ChatGPT** — Settings → Connectors → add a custom connector with the server URL
`https://mcp.gotcashback.com`, authentication OAuth. Availability of custom connectors varies by plan.

**VS Code (GitHub Copilot agent mode)** — run *MCP: Add Server* from the command palette, or add to
`.vscode/mcp.json`:

```json
{
  "servers": {
    "gotcashback": {
      "type": "http",
      "url": "https://mcp.gotcashback.com"
    }
  }
}
```

**Cursor, Windsurf and other clients** — most accept a JSON entry pointing at the remote server
(for Cursor: `~/.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "gotcashback": {
      "url": "https://mcp.gotcashback.com"
    }
  }
}
```

Nothing is installed or hosted on your side: it is a remote server, and every user connects to the
same URL.

## Example prompts

- "What's the best cashback rate for Best Buy right now?"
- "Are there any discounted gift cards for Target?"
- "What's the best cashback rate for Walmart in Canada?"
- "Compare gift card discounts for Home Depot."
- "Which cashback portals are available in the UK?"
- "Where can I buy Adidas products with the best cashback?"
- "Where can I buy dog food with the biggest discount?"
- "Which UK cashback portals pay via PayPal, and which one has the lowest minimum payout?"
- "Does TopCashback offer a sign-up bonus, and how often does it pay out?"
- "Alert me when Nike's cashback rate reaches 10%." *(account tool, prompts for sign-in)*
- "Add Best Buy to my favorite stores." *(account tool, prompts for sign-in)*

## How an assistant picks a tool

The server's initialize instructions tell connected assistants which tool answers which question, so
a plain question goes straight to the right lookup without any preliminary listing.

| User asks… | Tool |
|---|---|
| Cashback for a named store ("best cashback for Walmart", "Best Buy in-store cashback", "compare Expedia cashback portals") | `get_cashback_rates_by_store_name` |
| Discounted gift cards for a named store ("best Gap gift card discount") | `get_gift_cards_by_store_name` |
| Where to buy a brand's products with the most savings ("best cashback on Adidas products") | `get_best_deals_by_brand` |
| Where to buy a type of product with the most savings ("cheapest dog food with cashback") | `get_best_deals_by_category` |
| A portal's sign-up bonus, minimum payout, payment schedule or payout methods | `get_portals_by_name` / `get_portals` |
| Which stores offer cashback in a country | `get_stores_by_country` |
| Which countries are covered | `get_countries` |
| Favorites and alerts ("add Best Buy to my favorites", "alert me when Nike hits 10%") | `toggle_favorite_store` / `set_store_alert` |

```
User intent
 ├─ Cashback rates or gift card discounts?
 │    ├─ Cashback  ──► store name known? ──► get_cashback_rates_by_store_name
 │    │                store_id known?   ──► get_cashback_rates_by_store_id
 │    └─ Gift card ──► store name known? ──► get_gift_cards_by_store_name
 │                     store_id known?   ──► get_gift_cards_by_store_id
 ├─ A brand's products / a product category, not a store? ──► get_best_deals_by_brand / get_best_deals_by_category
 ├─ About a portal itself (bonus, payout, methods)?        ──► get_portals_by_name / get_portals
 ├─ Browse stores in a country?                            ──► get_stores_by_country
 └─ Country specified? ──► add country_code (lowercase ISO; 'gb' for the UK). Otherwise omit it.
                       ──► Return the offers, compare them, and show every tracked link.
```

The by-name tools search directly. Assistants should **never** call `get_stores_by_name`,
`get_stores_by_country` or `get_countries` first to find a store or its id, and should use the
`*_by_store_id` tools only when a `store_id` is already in hand from an earlier result.

## Tools

The server exposes tools only (no resources or prompts). Every tool declares `title`,
`readOnlyHint`, `idempotentHint`, `destructiveHint` and `openWorldHint`; the comparison tools also
publish an `outputSchema` and return typed `structuredContent`.

### Comparison tools (no sign-in)

All read-only, idempotent and anonymous.

| Tool | Title | Parameters | Returns (`structuredContent`) |
|---|---|---|---|
| `get_cashback_rates_by_store_name` | Cashback rates by store name | `store_name` (required), `country_code` | `{ stores: Store[] }` with `cashback_rates` |
| `get_cashback_rates_by_store_id` | Cashback rates by store ID | `store_id` (required) | `{ store: Store }` with `cashback_rates` |
| `get_gift_cards_by_store_name` | Gift card discounts by store name | `store_name` (required), `country_code` | `{ stores: StoreGiftCards[] }` |
| `get_gift_cards_by_store_id` | Gift card discounts by store ID | `store_id` (required) | `{ gift_cards: GiftCard[] }` |
| `get_best_deals_by_brand` | Best deals for a brand | `brand_name` (required), `country_code` | `{ matches: DealsMatch[] }` (up to 5 brands) |
| `get_best_deals_by_category` | Best deals for a category | `category_name` (required), `country_code` | `{ matches: DealsMatch[] }` (up to 5 categories) |
| `get_stores_by_name` | Search stores by name | `store_name` (required), `country_code` | `{ stores: Store[] }` (identity only, no rates) |
| `get_stores_by_country` | List stores in a country | `country_code` (required) | `{ stores: Store[] }` (no rates; can be hundreds of stores) |
| `get_portals` | List cashback portals | `country_code` | `{ portals: Portal[] }` with payout terms |
| `get_portals_by_name` | Search portals by name | `portal_name` (required), `country_code` | `{ portals: Portal[] }` |
| `get_portal_by_id` | Portal by ID | `portal_id` (required) | `{ portal: Portal }` |
| `get_countries` | Supported countries | — | `{ countries: Country[] }` |

### Account tools (OAuth 2.1 sign-in)

An anonymous call to any of these returns HTTP 401 with a `WWW-Authenticate` challenge pointing at
`/.well-known/oauth-protected-resource`, which spec-compliant clients turn into a sign-in prompt.
The comparison tools keep working without signing in.

| Tool | Title | Parameters | Scope | Annotations |
|---|---|---|---|---|
| `get_my_profile` | My profile | — | `profile` | read-only |
| `get_my_favorite_stores` | My favorite stores | — | `favorites` | read-only |
| `toggle_favorite_store` | Add or remove a favorite store | `store_id`, `favorite` (both required) | `favorites` | write, non-destructive |
| `get_my_alerts` | My alerts | — | `alerts` | read-only |
| `set_store_alert` | Set a store alert | `store_id`, `alert_type`, `threshold_percent` (all required) | `alerts` | write, non-destructive |
| `remove_store_alert` | Remove a store alert | `store_id`, `alert_type` (both required) | `alerts` | write, **destructive** |

The reads return `{ profile: UserProfile }`, `{ stores: FavoriteStore[] }` and `{ alerts: Alert[] }`;
the writes return a confirmation sentence. `alert_type` is the same string on the way in and on the
way out (`cashback` or `gift_card`), so a value read from `get_my_alerts` goes straight into
`set_store_alert` or `remove_store_alert`.

### Parameters

- **`store_name`** — the retailer or brand name as a shopper says it: `Walmart`, `Nike`, `Best Buy`,
  `Expedia`. Case-insensitive match at word boundaries against store names and known alternate
  names, best match first. Plain name only (no "cashback", "gift card", or country words).
- **`store_id`** — GotCashback's numeric id from any earlier result. Only for the `*_by_store_id`,
  favorite and alert tools.
- **`country_code`** — optional (required only for `get_stores_by_country`). Lowercase ISO 3166-1
  alpha-2: `us`, `ca`, `de`, `au`, `fr`, … **The United Kingdom is `gb`, not `uk`.** Omit it to search
  every supported country; the same retailer appears once per country with country-specific rates.
- **`portal_name`** / **`portal_id`** — a portal name (`Rakuten`, `TopCashback`) or the `id` from a
  portal result.
- **`brand_name`** / **`category_name`** — a product brand (`Adidas`) or category (`dog food`);
  multi-word phrases fall back to matching individual words.
- **`alert_type`** — `cashback` or `gift_card`. **`threshold_percent`** — greater than 0 and at most
  100 (10 means 10%). **`favorite`** — `true` to add, `false` to remove.

### Response fields

Every property carries a `description` in the tool's `outputSchema`; the essentials:

- **Store**: `store_id`, `name`, `country_code`, `url` (GotCashback store page), `domain`,
  `has_gift_cards` + `gift_cards_url` (by-country lists), `cashback_rates[]` (cashback tools).
- **Cashback rate**: `portal` (who offers it), `type` (`percent` or `fixedamount`), `rate`, `upto`
  (true = "up to X%", varies by category), `location` (`online` or `in-store`), `url` (tracked
  click-through that activates the cashback).
- **Gift card**: `seller`, `type` (`Digital` / `Physical`), `value_type` (1 = fixed `value`/`price`,
  2 = ranges `value_from`…`value_to` / `price_from`…`price_to`), `discount_percent` (off face value),
  `count`, `url` (tracked purchase link). The store wrapper adds `gift_cards_url`.
- **Portal**: `id`, `name`, `country_code`, `url`, `sign_up_bonus { amount, url }`, `minimum_payout` +
  `minimum_payout_currency`, `payment_frequency`, `payment_methods[]`.
- **Deals**: `name` (matched brand/category), `stores[]` ranked by `best_cashback_rate.rate` then
  `best_gift_card_discount_percent`, each with `url` and `gift_cards_url`.
- **Country**: `code` (pass as `country_code`), `name`, `localized_name`.
- **Alert**: `alert_id`, `store_id`, `store_name`, `country_code` + `country_name`, `alert_type`
  (`cashback` / `gift_card`), `threshold_percent`, `active`, `created_on`, `last_notified_on` +
  `last_notified_value` (absent until the alert first fires).
- **Favorite store**: `store_id`, `name`, `country_code`, `country_name`.
- **User profile**: `user_id`, `first_name`, `email`, `avatar`.

The comparison tools carry the API's raw JSON in their text block and the same data, wrapped in the
object above, as `structuredContent`. A lookup by an unknown id returns an `isError` result
("Store with id N not found."); an API outage returns an `isError` result asking the client to retry
with the same arguments; an empty search is a normal result with an empty list.

## Tracked links and presentation rules

The initialize response carries instructions that clients inject into the model's context:

- List **every** returned portal's rate, not only the best one, as a table with the portal, rate,
  online/in-store and a link column, then call out the best online and the best in-store rate. Do the
  same for gift cards (every seller).
- Always show each result's `url` (and `gift_cards_url` when present) as a clickable link. These are
  tracked GotCashback redirects; cashback and gift card discounts are only credited when the user
  clicks through them. Never show rates, stores or gift card prices without their links.
- Use the portal's payout terms (`sign_up_bonus`, `minimum_payout`, `payment_frequency`,
  `payment_methods`) to answer how and when a portal pays, and link a sign-up bonus through its own
  `url`; the bonus is only credited via that link.

If an assistant quotes a rate without a link, ask it for the link.

## Alerts

Alerts are the same ones you manage under Account → Alerts on the website.

- `alert_type` is `cashback` (the store's best cashback rate) or `gift_card` (the store's best gift
  card discount); `threshold_percent` is greater than 0 and at most 100.
- A cashback alert fires on the store's best **flat** rate from a cash-paying portal: "up to" tiered
  rates and airline-miles, credit-card and points portals are ignored, so the threshold is one you can
  actually earn.
- Re-saving an existing alert updates its threshold and re-arms it.
- Each account can have at most 10 active alerts; `set_store_alert` says so when the limit is reached.
- Notifications are sent by email.

## Authentication

- Protected resource metadata: https://mcp.gotcashback.com/.well-known/oauth-protected-resource
  (scopes `profile`, `favorites`, `alerts`, `offline_access`; bearer tokens in the `Authorization`
  header).
- Authorization server: https://www.gotcashback.com/ — metadata at
  https://www.gotcashback.com/.well-known/oauth-authorization-server (also served as
  `/.well-known/openid-configuration`).
- Flow: authorization code with PKCE (S256 required) plus refresh tokens. Clients that request
  `offline_access` receive refresh tokens, so the connection stays signed in. Access tokens are valid
  for 30 days and refresh tokens for 90 days.
- Dynamic client registration (RFC 7591): `POST https://www.gotcashback.com/connect/register`,
  anonymous and rate-limited. `token_endpoint_auth_method` may be `none` (public clients),
  `client_secret_post` or `client_secret_basic` (confidential clients); the server metadata also lists
  `private_key_jwt`. Up to 10 redirect URIs: `https` URIs, `http` on loopback hosts (`localhost`,
  `127.0.0.1`, `[::1]`) for local agents such as Claude Code, or private-use schemes such as
  `com.example.app:/callback` for native apps. Every registered client is granted all scopes.
- End-user sign-in is GotCashback's own social login, the same as on the website. No registration
  or API key is needed for the comparison tools.
- Calling an account tool without a bearer token returns HTTP 401 with a `WWW-Authenticate` challenge
  that references the protected resource metadata; clients open the sign-in page, the user approves
  the requested scopes, and the tool call completes.
- To disconnect, remove the connector from your client. If a tool later reports a missing scope,
  remove and re-add the connector so the client requests authorization afresh.
- Policy for agents: https://www.gotcashback.com/auth.md

| Scope | Grants |
|---|---|
| `profile` | Read your GotCashback profile (name, email, avatar) |
| `favorites` | Read and change your favorite stores |
| `alerts` | Read, create and remove your cashback and gift card alerts |
| `offline_access` | Refresh tokens so the connection stays signed in |

## Endpoints

| URL | What it is |
|---|---|
| https://mcp.gotcashback.com | MCP endpoint: Streamable HTTP, JSON-RPC over POST, stateless (no session affinity needed) |
| https://mcp.gotcashback.com/.well-known/mcp/server-card.json | MCP server card: server, transport, capabilities and authentication (also served by www.gotcashback.com) |
| https://mcp.gotcashback.com/.well-known/oauth-protected-resource | OAuth protected resource metadata |
| https://www.gotcashback.com/.well-known/oauth-authorization-server | Authorization server metadata |
| https://www.gotcashback.com/connect/register | Dynamic client registration |
| https://mcp.gotcashback.com/health | Health check |
| https://www.gotcashback.com/auth.md | Agent authentication policy |
| https://www.gotcashback.com/llms.txt | Agent-readable site index (every store, gift card, brand, category and portal page has a Markdown mirror) |
| https://api.gotcashback.com/swagger/ | The free REST API behind the tools (OpenAPI, no key required) |

CORS is open to any origin, so browser-based agents can connect directly; the `Mcp-Session-Id` and
`WWW-Authenticate` response headers are exposed.

List the tools without a client:

```bash
curl -s -X POST https://mcp.gotcashback.com/ \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## Troubleshooting

- If a connection fails, check that the URL is exactly `https://mcp.gotcashback.com` with no trailing
  path.
- If an account tool keeps asking you to sign in, remove and re-add the connector.
- Rates are refreshed by scheduled jobs several times a day, not in real time; treat any rate older
  than a day as stale and ask again.

## Links

- Human documentation: https://www.gotcashback.com/mcp-server/
- Changelog: [CHANGELOG.md](CHANGELOG.md)
- Cashback widget for websites: https://www.gotcashback.com/widget/
- Telegram bot: https://t.me/GotCashbackBot
- Privacy policy: https://www.gotcashback.com/Privacy/ · Terms: https://www.gotcashback.com/Terms/
- Support: support@gotcashback.com
