# In-play Polymarket football: position audit, live adjustment, and PnL lessons（2026-06-13）

## Session learning

This session exposed three durable workflow lessons for World Cup / football Polymarket work:

1. **Separate planned pool ledger from real account trades.**
   - Local pool allocation files can contain intended or proportional exposure, but real PnL must come from account data.
   - When the user asks for “真实盈亏 / account1 持仓和历史记录”, query Polymarket data-api `positions` and `activity/trades` for the actual `proxyWallet`.
   - If the local ledger says a match exists but account activity has no matching slug/title/asset, explicitly say it is not an account-verified trade.

2. **For live match advice, verify orderbook before recommending action.**
   - Use `positions` to identify exact `asset` token IDs.
   - Use `https://clob.polymarket.com/book?token_id=<asset>` to get best bid/ask.
   - Use best bid as the conservative “can sell now” value.
   - Do not rely only on old entry cost or user-reported price if live orderbook is queryable.

3. **For PnL, combine realized cashflow and remaining mark value.**
   - For a fully sold position: `PnL = sum(SELL usdcSize) - sum(BUY usdcSize)`.
   - For a partially sold position: `PnL = sum(SELL usdcSize) + currentValue_of_remaining_position - sum(BUY usdcSize)`.
   - For open positions: `PnL = currentValue - initialValue`, preferably valued at CLOB best bid when available.
   - Keep “positions.cashPnl” as a useful cross-check, but activity cashflow gives a clearer audit trail when there were partial sells.

## Concrete account1 examples from the session

The user asked to audit account1. Data-api for proxyWallet `0x912c...F468` showed:

- `Will United States win on 2026-06-12? / Yes`
  - BUY `usdcSize=609.36`
  - SELL `usdcSize=1237.5`
  - realized PnL `+628.14`
- `United States vs. Paraguay: O/U 2.5 / Under`
  - BUY `usdcSize=404.55919`
  - partial SELL `usdcSize=78.06048`
  - remaining `positions.currentValue=1.2263`
  - current PnL `78.06048 + 1.2263 - 404.55919 = -325.27241`
- `France win 2026 World Cup / Yes`
  - BUY total `0.9903`
  - current value about `0.96`
  - current PnL about `-0.0303`

Therefore audited account1 World Cup/FIFA PnL at that moment was about `+302.8373 USDC`.

The same audit found **no Canada vs Bosnia activity or position in account1**, despite a prior conversational/hypothetical PnL calculation. Future agents must not carry that hypothetical result into real account accounting.

## In-play decision heuristics reinforced

### Canada 0-1 Bosnia at 58'/75'

- Do not chase the pre-match favorite after it falls behind.
- Reduce or hedge the favorite win position if it remains large.
- Under 2.5 can remain a recovery line while total goals remain low, but must be reevaluated after any equalizer.

### Canada 1-1 at 78'

- Under 2.5 loses its safety margin: only one more goal kills it.
- If holding full Under exposure, sell a substantial portion or at least reassess against live best bid.
- Do not buy Over just to emotionally hedge unless the price/value calculation supports it.

### USA 2-0 Paraguay at 27'

- USA win position becomes high-probability and can be partially sold to lock profit.
- Under 2.5 becomes fragile very early: with 60+ minutes remaining, one more goal kills it.
- Query orderbook; in the session USA Yes had best bid around `0.93`, Under 2.5 around `0.17`.
- A balanced adjustment was: sell most/all Under, optionally sell 30%-50% of USA Yes to lock profit while leaving upside.

## Reporting pattern for future sessions

When user asks “现在怎么操作” during live match:

1. State conclusion first: hold / reduce / sell / no chase.
2. Show current verified positions: market, size, cost, current best bid/ask.
3. Give 2-3 scenarios with approximate PnL/risk.
4. Do not say “执行” unless the user explicitly asks to place/close orders.
5. If user later asks “真实盈亏”, audit against activity and positions, not the earlier tactical scenario math.
