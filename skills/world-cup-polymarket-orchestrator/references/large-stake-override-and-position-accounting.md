# Large-stake override and position accounting

Use this reference when the user overrides normal World Cup/Polymarket stake sizing (e.g. “坚持下 1000”, “每场 250”, “补到总金额 500”).

## 1. Treat the user override as explicit authorization, but still report the risk

- If the requested stake exceeds the skill’s normal unit sizing, warn once, briefly, with the safer alternative.
- If the user then explicitly confirms the larger size, execute the confirmed size; do not keep debating.
- Restate that Polymarket settles in USDC even if the user says USD/USDT.

## 2. Convert “total to X” into a top-up amount

When the user says “把某场提高到总金额 X”:

1. Query current positions for all relevant markets in that match.
2. Compute current match exposure/cost before top-up.
3. `top_up = target_total - current_total`.
4. Allocate top-up across the same directions either:
   - by the existing cost ratio, if the user did not specify a new split; or
   - by the latest explicit split, if the user specified one.
5. After execution, report both:
   - planned/order notional total, and
   - Polymarket positions API current `initialValue`/`currentValue` total.

Note: positions API may lag or show rounded/zero values briefly after a fresh order. Re-query after 10–20 seconds and, if needed, cross-check `data-api.polymarket.com/activity`.

## 3. Position accounting: use two views and label them

### A. Order/activity cash-flow view

Best for realized or just-executed orders.

- Buy cost = sum BUY `usdcSize` or order `makingAmount` depending endpoint semantics.
- Sell proceeds = sum SELL `usdcSize` or order `takingAmount` depending endpoint semantics.
- Realized-style PnL for a closed direction = sell proceeds - buy cost.

Use this when the user asks: “这场成本多少、盈利多少” after a position was sold or resolved.

### B. Positions API mark-to-market view

Best for open positions.

- Open cost = `initialValue`.
- Current mark = `currentValue`.
- Floating PnL = `cashPnl`.
- Current probability/price = `curPrice`.

Use this when the user asks: “目前持仓多少”.

### Always label discrepancies

It is normal for the order response, activity feed, and positions API to differ slightly because of fees, partial precision, index lag, and endpoint semantics. Do not silently mix them. Say:

- “按下单计划口径…”
- “按 Polymarket positions 当前回读…”
- “按 activity 成交流水…”

## 4. Verify every live execution

Before buying/selling:

1. Execute only from HK server (`<YOUR_HK_SERVER_IP>`, `/opt/polymarket-worldcup`).
2. Verify `country == HK`.
3. Resolve fresh Gamma market and outcome token; never reuse guessed or stale token IDs.
4. Check CLOB orderbook for best ask/bid.
5. Check balance/allowance.

After execution:

1. Save a JSON log under `/opt/polymarket-worldcup/logs/`.
2. Report order ID and tx hash for each matched order.
3. Re-query positions after 10–20 seconds.
4. If positions API misses a fresh position, verify by activity feed and re-query.

## 5. Suggested reporting format for this user

Keep it short and operational:

```text
已执行，全部 matched。

Match A:
- Direction 1: amount / price / size / orderID
- Direction 2: amount / price / size / orderID
- Match subtotal: cost / current value / floating PnL

Match B:
...

Total:
- Cost: ... USDC
- Current value: ... USDC
- Floating PnL: ... USDC
```

Avoid long theory after execution; the user wants numbers and confirmation.
