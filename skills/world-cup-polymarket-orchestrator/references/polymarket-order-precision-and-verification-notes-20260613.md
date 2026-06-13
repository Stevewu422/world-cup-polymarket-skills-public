# Polymarket order precision + verification notes (2026-06-13)

Use when placing larger football/World Cup orders through `pm_worldcup.py` or direct `py_clob_client_v2` wrappers.

## Precision pitfall: maker/taker amount decimals

A live BUY can fail with:

```text
invalid amounts, the market buy orders maker amount supports a max accuracy of 2 decimals, taker amount a max of 4 decimals
```

This is not a market/auth failure. It means the selected `price * size` or size precision violates CLOB precision rules.

### Fix pattern

1. Keep `price` at the current best ask / intended limit.
2. Adjust `size` so:
   - maker/notional amount (`price * size`) is clean to 2 decimals; and
   - taker/share amount has acceptable precision.
3. Prefer integer share sizes for large FOK BUYs when possible.
4. Recompute actual notional and report the small difference from the user's round budget.

Example from Qatar vs Switzerland:

```text
Target: 1300 USDC at 0.81
Rejected size: 1604.94 (notional 1300.0014)
Accepted size: 1604 (notional 1299.24)

Target: 700 USDC at 0.58
Accepted size: 1207 (notional 700.06)
```

## Orderbook verification pitfall

If a SDK orderbook helper unexpectedly returns empty bids/asks, do not conclude the market has no liquidity until checking the raw CLOB endpoint:

```text
https://clob.polymarket.com/book?token_id=<clobTokenId>
```

Use raw `/book` best bid/ask as the conservative execution value when SDK serialization looks suspicious.

## Reporting nuance

After execution, verify with both:

- CLOB order response: `status`, `orderID`, `transactionsHashes`, `makingAmount`, `takingAmount`.
- Data API positions/activity: market title, outcome, size, avgPrice/price, initialValue/currentValue, tx hash.

For clean cost basis, prefer:

1. `positions.initialValue` after indexing; or
2. CLOB `makingAmount` immediately after order response.

`activity.usdcSize` can differ slightly from CLOB notional; mention it only as an activity-display value, not the primary cost basis.
