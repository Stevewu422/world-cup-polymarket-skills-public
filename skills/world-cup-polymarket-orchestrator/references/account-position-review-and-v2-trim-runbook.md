# Account position review and v2 trim runbook

Use when the user asks: “看一下 account1/account2 当前持仓，给出修改意见 / 调仓建议 / 风险复核”.

## Goal

Produce a grounded position review before suggesting any trade. The deliverable is advice first, not execution. Do **not** place SELL/BUY orders unless the user separately authorizes exact execution.

## Required readback

1. Query wallet cash/balance.
2. Query `data-api.polymarket.com/positions?user=<proxyWallet>&limit=...` for current open positions.
3. Query recent `data-api.polymarket.com/activity?user=<proxyWallet>&limit=...` and search for combo/conditional titles such as `A AND B`.
   - Some combo markets may not appear in ordinary positions.
   - If CLOB `/book?token_id=<comboAsset>` returns `No orderbook exists`, record it as “combo value needs platform/portfolio display or settlement; do not silently drop it.”
4. For each open position record:
   - title / slug / outcome
   - size
   - avgPrice
   - initialValue
   - currentValue
   - cashPnl / percentPnl
   - current best bid/ask from CLOB for conservative executable value
5. Sum normal positions by `initialValue` and `currentValue`; then add known combo cost/value separately if available.

## v2 risk interpretation

Apply the current `sports-betting-risk-management` v2 lens:

- If current prices do not clear the executable edge gate (`q - ask >= 5pp`), mark “no new add”.
- Compare open cost to current bankroll/pool basis:
  - 15–25% open exposure = aggressive but usually tolerable.
  - >25% = stop new adds and consider trimming.
  - >50% = materially overexposed; trim secondary/correlated markets first.
- Tag directional concentration: `CHALK`, `UNDER`, `OVER`, `HANDICAP`, `COMBO`.
- Same-direction cross-match exposure should be highlighted; do not only look at per-match caps.

## Trim priority

Default trim order when exposure is high and no fresh edge exists:

1. Optional correlated secondary markets first:
   - spreads / handicaps such as `-1.5`, `-2.5`
   - totals that were downgraded from main thesis to protection/observation
   - BTTS / exact score / first-half props
2. Preserve core side/win markets if still directionally valid.
3. Keep small hedges/protection positions when they reduce the largest core thesis risk, unless they have become stale or oversized.
4. Only cut main side/win markets after secondary risk is reduced, or when total open exposure remains too high.

## Advice format

Report in Chinese, conclusion first:

```text
结论：当前总敞口偏重/正常；建议停止新增，只减不加/继续持有。

当前持仓：
- Market A: size / cost / current value / PnL / bid-ask
...

风险判断：
- 总开放成本 vs 本金池：...%
- 同方向集中：...
- combo/conditional 仓位是否已单独识别：...

修改建议：
方案一：平衡减仓（先减副仓，不动主仓）
- 卖 ...%，预计回收 ...

方案二：严格 v2 风控（进一步降主仓）
- 卖 ...%，预计回收 ...

未执行说明：以上只是建议；若要执行，需重拉 orderbook，价格超过 15 分钟必须重拉。
```

## Pitfalls

- Do not add positions during a review task unless the user explicitly says to execute.
- Do not use stale earlier allocation files as current positions; query live positions/activity.
- Do not sum shares across different tokens; only USD notional/current value can be summed.
- Do not ignore combo/parlay activity just because it is missing from ordinary positions.
- Do not treat a small mark-to-market profit as proof of edge; use the v2 edge gate and CLV/Brier rules.
