# Large add-position execution: precision, guardrails, and market-selection notes

Session date: 2026-06-13

## Trigger

User instructed account1 to increase Switzerland match exposure by 50% and Brazil match exposure by 50%.

## Durable lessons

### 1. Interpret “match +50%” through current strategy, not mechanically every token

- Switzerland match had two active intended markets: Switzerland win and Switzerland -1.5. Add 50% proportionally to both.
- Brazil match had Brazil win plus Under 2.5, but Under 2.5 had just been downgraded to a protection position after injury-lineup reassessment. The 50% add was therefore applied only to the main direction, Brazil win, not to the downgraded Under.

Rule: if a market was explicitly downgraded to “protection/观察仓”, do not add to it under a broad “match add” instruction unless the user explicitly names that market.

### 2. Temporary max-order override without editing `.env`

The helper enforces `POLYMARKET_MAX_ORDER_USD=10` by default. For user-authorized large orders, avoid editing `.env`; set a per-command environment override:

```bash
POLYMARKET_MAX_ORDER_USD=700 python3 pm_worldcup.py place ...
```

This preserves the default guardrail for future commands.

### 3. BUY amount precision pitfall

A BUY order with size decimals such as `792.22` can fail with:

```text
invalid amounts, the market buy orders maker amount supports a max accuracy of 2 decimals, taker amount a max of 4 decimals
```

Practical fix used successfully:

- round BUY `size` to an integer share amount for larger orders;
- keep price at valid cent precision;
- rerun as FOK.

Example executed successfully:

```bash
POLYMARKET_MAX_ORDER_USD=700 python3 pm_worldcup.py place \
  --token-id <Switzerland-win-token> --side BUY --price 0.82 --size 792 \
  --order-type FOK --live --yes-i-understand-real-money
```

### 4. Verification after execution

After matched orders:

1. Re-query positions for affected markets.
2. Confirm expected size/initialValue increased.
3. Query recent activity for order hashes.
4. Record per-market notional and participant allocations using the pool snapshot active at the order time.

## Example execution result

```text
Switzerland win: BUY 792 @ 0.82, notional 649.44, matched
Switzerland -1.5: BUY 593 @ 0.59, notional 349.87, matched
Brazil win: BUY 826 @ 0.59, notional 487.34, matched
Total added: 1486.65 USDC
```

## Pitfall

Do not conflate Polymarket `usdcSize` in activity with the clean limit-price notional used for ledger allocation. For participant allocation, use the intended/matched notional from `price * size` unless the user explicitly wants fee-inclusive platform activity totals.