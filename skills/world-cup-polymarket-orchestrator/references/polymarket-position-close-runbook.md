# Polymarket position close / sell-to-close runbook

Use when the user asks to close / flatten / 平掉 a Polymarket position, especially football O/U markets.

## Preconditions

- Treat close/flatten as a real-money action.
- Execute only from the configured HK server/workdir for Polymarket live actions.
- Verify HK exit before posting any order.
- Re-fetch the current Gamma market and CLOB token; do not rely only on a screenshot or old token.
- For O/U 2.5 football markets, confirm the full-match market title exactly, because there may also be:
  - full match `O/U 2.5`
  - `1st Half O/U 2.5`
  - `2nd Half O/U 2.5`
  - team total O/U 2.5

## Sell-to-close workflow

1. Resolve the target market and outcome:
   - Example full match: `Korea Republic vs. Czechia: O/U 2.5`
   - Outcome to close: existing `Under` asset token.
2. Query current positions by wallet via Polymarket Data API and locate the exact `asset` token.
3. Record position before:
   - size
   - avgPrice
   - initialValue
   - curPrice/currentValue
4. Query CLOB orderbook for the exact asset.
5. For immediate flattening, sell the entire size at the current best bid.
   - If the user asked to close now, prioritize execution over trying to improve one tick unless they asked for a limit price.
6. Post a `SELL` order for the exact held size.
7. Report only non-sensitive fields:
   - market / outcome
   - size
   - limit price
   - status
   - orderID
   - transaction hash
   - approximate USDC received / realized PnL when available

## Verification pattern

Do not rely only on the immediate post-order response or one immediate positions call.

After `status=matched`:

1. Wait a few seconds.
2. Query Data API `positions` with a cache-busting parameter such as `_=<unix_ts>`.
3. Confirm the target asset no longer appears, or `size` is zero.
4. Query Data API `activity` for recent trades and confirm:
   - `side=SELL`
   - target `asset` token
   - target market title
   - expected size
   - transactionHash exists
5. If the first positions call still shows the old position, retry after 8-15 seconds before declaring failure; the Data API can be briefly stale.

## Pitfalls

- Do not accidentally close a half-time, second-half, or team-total Under when the user means full-match Under 2.5.
- Do not sum shares across different markets; report shares by market/outcome and only aggregate USDC exposure if needed.
- If positions appear unchanged immediately after a matched order, verify against recent `activity` and retry positions with cache busting before taking another action.
- Do not expose private keys, API secrets, passphrases, or raw `.env` contents in chat or logs.