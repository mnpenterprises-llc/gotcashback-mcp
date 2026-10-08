# Changelog

The hosted server at `https://mcp.gotcashback.com` is deployed continuously; the version number is
the one reported in `serverInfo` and the server card, and is bumped when the registry listing is
republished. Dates are the dates the change went live.

## 1.1.0 — 2026-09-04 (Official MCP Registry publish)

Listed as `com.gotcashback/gotcashback` on https://registry.modelcontextprotocol.io.

### Changes since the publish (server still reports 1.1.0)

- **2026-09-08** — Cashback alerts fire only on a store's best **flat** rate from a cash-paying
  portal. "Up to" tiered rates and airline-miles, credit-card and points portals no longer satisfy a
  threshold; alerts last notified on an excluded rate re-arm. The `set_store_alert` description states
  the rule.
- **2026-09-07** — Store-name search (`get_cashback_rates_by_store_name`, `get_gift_cards_by_store_name`,
  `get_stores_by_name`) matches at word boundaries instead of anywhere in the name, so "Gap" no longer
  matches every store containing those letters. Best match still comes first.
- **2026-09-06** — The account reads (`get_my_profile`, `get_my_favorite_stores`, `get_my_alerts`)
  return the same snake_case field names as the comparison tools (`store_id`, `alert_type`, …) in both
  the text block and `structuredContent`; `alert_type` round-trips as `cashback` / `gift_card`, so a
  value read from `get_my_alerts` can be passed straight to `set_store_alert` / `remove_store_alert`.
- **2026-09-05** — The server description and initialize instructions quote the live coverage
  figures (portals, countries, US portals) from `/v1/coverage` instead of a fixed "30+ countries".
- **2026-09-04** — Server-side telemetry (Application Insights) for availability monitoring. No
  change to tool behaviour.

### What 1.1.0 shipped (2026-08 to 2026-09-04)

- Optional OAuth 2.1 sign-in (authorization code + PKCE, dynamic client registration, refresh tokens)
  and the six account tools: `get_my_profile`, `get_my_favorite_stores`, `toggle_favorite_store`,
  `get_my_alerts`, `set_store_alert`, `remove_store_alert`. Anonymous calls to them return a 401
  challenge; the comparison tools stay anonymous.
- Portal payout terms (`sign_up_bonus`, `minimum_payout`, `payment_frequency`, `payment_methods`) on
  every portal result, with the sign-up bonus linked through its own tracked `url`.
- `get_best_deals_by_brand` and `get_best_deals_by_category`.
- Tracked GotCashback links (`url`, `gift_cards_url`) on every store, rate, gift card and portal
  result; the initialize instructions require clients to show every returned portal with its link,
  not only the best rate.
- Tool metadata tuned for discovery: titles, when-to-use descriptions, read-only / idempotent /
  destructive annotations and published output schemas with typed `structuredContent`.
- Stateless Streamable HTTP transport; SEP-1649 server card at
  `/.well-known/mcp/server-card.json`.

## 1.0.x — 2026-02 to 2026-07

- 2026-02-25 — First release: cashback rates by store id, stores by country, countries, portals.
- 2026-02-26 — Gift cards by store id; cashback rates by store name.
- 2026-03-03 — Gift cards by store name; portals by name.
- 2026-07-27 — Server card and agent-discovery metadata.
