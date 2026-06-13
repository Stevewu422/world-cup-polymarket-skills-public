# Polymarket position close + risk-override runbook

Use this reference when the user asks to inspect, close/flatten, or massively increase an existing World Cup Polymarket position.

## 1. Read-only position inspection

For a specific market/outcome:

1. Resolve the current `clobTokenId` from Gamma, not screenshots or old logs.
2. Query `data-api.polymarket.com/positions?user=<proxyWallet>&limit=500` and filter by `asset == clobTokenId`.
3. Report separately:
   - `size` / shares
   - `avgPrice`
   - `initialValue` (cost basis)
   - `currentValue`
   - `cashPnl` / `percentPnl`
   - whether the position is still open.
4. If asking about a whole match, list each independent market separately; do not add share counts across 1X2, totals, spreads, or props. It is OK to sum USDC cost/current value/PnL across markets.

## 2. Closing / flattening a position

When the user says “平掉 / close / flatten / 现价平”:

1. Treat this as explicit authorization to sell the current position for that specified market/outcome only.
2. Still perform the execution checklist:
   - execute only from HK server `<YOUR_HK_SERVER_IP>:/opt/polymarket-worldcup`;
   - verify `country == HK`;
   - verify `active=true`, `closed=false` when relevant;
   - query current orderbook for the exact token.
3. Use the best bid for immediate sell-to-close unless the user specifies a limit.
4. Sell only the current open `size` for that asset; round conservatively to supported precision.
5. Save a non-secret log under `/opt/polymarket-worldcup/logs/`.
6. Verification must use both:
   - `positions` filtered by asset; and
   - `activity` filtered by asset/order, because `positions` may lag briefly right after `matched`.
7. Final report must include:
   - market / outcome;
   - sell size;
   - sell price;
   - orderID;
   - transaction hash;
   - matched/open/failed status;
   - post-close position size.

### Important data-source nuance

`post_order` response fields can differ from data-api activity accounting:

- `takingAmount` / `makingAmount` in the order response are useful execution receipts.
- `data-api activity.usdcSize` is often the better normalized value for realized PnL summaries.

When computing realized PnL after a close, prefer:

```text
realized PnL = sum(activity SELL usdcSize) + currentValue - sum(activity BUY usdcSize)
```

If a position is fully closed, `currentValue = 0`.

## 3. Match-level PnL after closing

For an already closed match, calculate per market:

```text
cost = sum(BUY usdcSize)
recovered = sum(SELL usdcSize)
profit = recovered - cost
ROI = profit / cost
```

Then provide a match-level USDC total:

```text
total cost = sum(market costs)
total recovered = sum(market recovered)
total profit = sum(market profit)
```

Do not aggregate shares across different markets.

## 4. Oversized stake / risk-override handling

If the user proposes a stake far above the normal staking plan (example: “给你 1000 USD 去下注这场”):

1. Do not immediately execute just because a number was given.
2. Compare proposed notional to:
   - available USDC balance;
   - existing exposure in the same day/match;
   - normal rule: single match usually <= 2份 and daily drawdown stop = 5份.
3. Push back plainly if it violates risk discipline.
4. Offer 2-3 safer alternatives, e.g.:
   - discipline plan (small notional);
   - aggressive but bounded plan;
   - user-forced full-risk plan.
5. Require a precise explicit confirmation before live execution when the stake materially exceeds the plan. The confirmation should include amount and split by market/outcome.
6. Never silently reinterpret “1000 USD” into a smaller live order; either propose a safer plan or execute only after explicit confirmation.

## 5. User-facing style

The user prefers concise Chinese, conclusion first, with TTS for betting/stock-style decisions. For Telegram:

- Report USDC cost/value/profit plainly.
- Include the `MEDIA:` TTS path when generated.
- Keep detailed logs on the server; do not paste secrets or private keys.
