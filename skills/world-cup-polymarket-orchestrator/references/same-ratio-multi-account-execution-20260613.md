# Same-ratio multi-account Polymarket execution notes (2026-06-13)

Use when the owner says phrases like “同样比例 / account2 也按同样比例 / 那这场 account2 500” after a prior Polymarket order plan or execution.

## Interpretation

- “同样比例” means mirror the most recent approved/executed allocation ratio for that match, scaled to the new total notional.
- Do **not** blindly reuse old token IDs from screenshots or scripts. Re-fetch or verify the current Gamma market tokens and CLOB orderbook for the target match/market.
- If the prior order was approximately `A market cost : B market cost`, compute the new target as:
  ```text
  new_A_notional = new_total * A_cost / (A_cost + B_cost)
  new_B_notional = new_total * B_cost / (A_cost + B_cost)
  ```
- Then convert notional to size using current limit prices, respecting Polymarket precision constraints.

## Precision / rounding pitfall

Polymarket CLOB can reject BUY orders with:

```text
invalid amounts, the market buy orders maker amount supports a max accuracy of 2 decimals, taker amount a max of 4 decimals
```

Practical fix:

- Use sizes that produce a `price * size` notional with at most 2 decimal places for the maker amount.
- Integer share sizes usually avoid the issue for normal 2-decimal prices.
- If exact notional is impossible, slightly undershoot/overshoot by less than a dollar and report the actual notional.

Examples from the session:

```text
1300 USDC at 0.81:
- size 1604.94 failed precision validation
- size 1604 succeeded; notional 1299.24

700 USDC at 0.58:
- size 1207 succeeded; notional 700.06

500 USDC same-ratio mirror of 599.72 : 349.80:
- Brazil win at 0.58 size 544 -> 315.52
- Under 2.5 at 0.55 size 335 -> 184.25
- total 499.77
```

## Account1 vs account2 execution

- account1 should use the standard `.env` path and `pm_worldcup.py` where possible.
- account2 may have a broken `.env.accounts/account2.env` (`401 Unauthorized/Invalid api key`) but still work via the uploaded/direct Python script path that derives creds on the fly.
- When using account2 direct script:
  - parse only required constants (`HOST`, `PRIVATE_KEY`, `FUNDER`, `CHAIN_ID`) from the saved script;
  - never print secrets;
  - verify HK egress;
  - verify each token orderbook best ask is <= intended limit;
  - place FOK orders;
  - verify with `positions` and `activity` for the account2 funder.

## Verification/reporting checklist

After execution, report:

- account used (`account1` / `account2`);
- market/outcome;
- price, size, exact notional;
- status (`matched`/open/failed);
- orderID and tx hash;
- post-order balance;
- positions readback (`size`, `avgPrice`, `initialValue`).

Prefer CLOB response `makingAmount` / positions `initialValue` as clean cost basis. `data-api activity.usdcSize` may differ slightly and should not replace the clean cost basis unless investigating PnL/indexing differences.
