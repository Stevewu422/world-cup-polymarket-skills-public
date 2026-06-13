# Polymarket Gamma API probe for World Cup match markets

Use this reference when producing match-by-match World Cup betting plans with clickable Polymarket links.

## Proven pattern

For a known Polymarket match slug, query Gamma directly instead of scraping the rendered page:

```text
https://gamma-api.polymarket.com/events?slug=<event-slug>
```

Example slugs observed for the 2026 opener:

```text
fifwc-mex-rsa-2026-06-11
fifwc-mex-rsa-2026-06-11-more-markets
```

Clickable pages:

```text
https://polymarket.com/sports/world-cup/<slug>
```

## What to extract

From the event JSON:

- `title`
- `endDate`
- `restricted`
- `volume`, `liquidity`
- `markets[]`
  - `question`
  - `active`, `closed`
  - `volume`
  - `outcomes`
  - `outcomePrices`

`outcomes` and `outcomePrices` may be JSON-encoded strings; parse them before presenting.

## Output discipline

For each market, report only actionable items:

- market title/question
- Yes/No or named outcome direction
- current probability/price if available
- execution threshold
- stake size
- pass/no-bet condition

Do not fabricate a market. If the slug query returns empty or no direct match exists, say `Polymarket：未找到该场直接市场` and optionally give a search or event-category link.

## First-match insight captured

Opening match markets may be emotionally priced around the host narrative. When the favorite win market is already hot, prefer conditional/value markets such as under totals or underdog + spread, and keep total exposure small.