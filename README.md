# Underkeel

Site: https://underkeel.xyz

Exit depth and liquidations of Morpho Blue markets on Base, refreshed every 6 hours. Method: Clearance v3 of 16 Nov 2026
(`method.html`).

Clearance at a size = break-even exit cost - exit cost, in percentage points. The break-even exit cost is
1 - 1/(1 + bonus): a liquidator repays 1, seizes 1 + bonus and breaks even when the sale loses bonus/(1 + bonus), a
little less than the bonus (11.25% at a 12.68% bonus). Positive clearance - room for a flash-loan liquidator at that
size; negative - a single liquidation of that size loses money. The clearing size is the largest measured size with
positive clearance. Quote-dependent debt is the amount borrowed in markets that clear some size only with RFQ quotes.
Before 16 Nov 2026 (v1, v2) the exit cost was set against the bonus itself; see the changelog in `method.html`.

- `exit-depth.html`, `data/exit-depth.json` - for every Morpho-listed market on Base with at least $100k borrowed: the
  liquidation bonus fixed by its LLTV, `min(15%, 1/(0.3 x LLTV + 0.7) - 1)`, and the share lost selling its collateral
  into the loan token at $25k, $200k and $1M (best KyberSwap route, market makers included, both sides valued by
  KyberSwap), and the collateral of positions with health factor below 1.1 and 1.25.
- `venues.html` - where each sale goes: in `data/exit-depth.json`, `venues` is the share of the collateral each
  KyberSwap source takes on the hops out of the collateral token (`kind`: `rfq` - off-chain RFQ quote or limit order,
  `maker_pool` - on-chain pool priced by a market maker, `vault` - vault redemption, `amm` - AMM pool), and
  `exit_no_rfq` is the exit cost with RFQ quotes excluded (`null` - no route without them). In `exit`, `exit_no_rfq`
  and `venues` a size is missing when it was not measured (the quote failed or there is no price for the token);
  `null` in `exit` - no route (a size above one with no route is not asked again and is `null` too). `gap`, when
  present, says why a market is not fully measured (`no_route`, `vault`, `no_price`, `api_failed`, `budget`);
  `exit_dex` - for a vault share of the loan token, KyberSwap's cost at the sizes left out of `exit` because its route
  skips the vault's redemption; `price_source: "oracle"` - the sale was sized at the market's oracle price (the Morpho
  API has no USD price for the token). A `near` band is `null` when its collateral has no USD value.
- `liquidations.html`, `data/liquidations.json` - Morpho Blue liquidations on Base over 30 days: bonus paid, the share
  that went through Chainlink SVR auctions, by day and by pair (every pair with a liquidation).
- Both JSON files carry `schema`, a format version that changes whenever the meaning of a field changes:
  `exit-depth.json` is 3 since 16 Nov 2026 (`near` may be `null`; vault shares as above), 2 from 13 Nov 2026 (files
  without `schema` are version 1, where `null` also stood for a failed quote); `liquidations.json` is 2 since 16 Nov
  2026 (`pairs` has every pair; 1 - the 10 with the most bonus). `exit-depth.json` also has `at_iso` (the snapshot time
  in ISO 8601) and `min_borrow_usd` (the selection threshold, USD).

`markets/<pair>-<lltv>.html` - one page per market: exit cost by size against the break-even exit cost, clearance,
venues, collateral near liquidation, the pair's other LLTVs.
`notes/` - dated notes on events in these markets (Atom feed `notes/feed.xml`).
`method.html` - how every number is measured and what is not measured; `data.html` - the fields of both JSON files;
`about.html` - who runs the site and the disclosure.

Every commit is one snapshot: the history of this repository is the history of exit depth.
