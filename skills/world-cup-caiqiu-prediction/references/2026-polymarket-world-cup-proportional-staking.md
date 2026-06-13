# 2026 World Cup Polymarket execution lessons

Session-derived notes for future World Cup/football betting tasks.

## User preference

- Do not allocate exactly one unit to every pick just because the user says “1份=10USD”.
- The user wants proportion differences that match the prediction confidence: larger for clearer/value picks, smaller for uncertain picks.
- Explain which picks are larger/smaller and why.

## Practical allocation pattern

Example for three-match slate with side + total markets:

```text
Mexico win: 1.1份
Mexico Under 2.5: 0.9份
Korea Republic win: 0.5份
Korea Republic Under 2.5: 0.7份
Canada win: 0.9份
Canada Under 2.5: 0.8份
```

Rationale:

- Mexico side was strongest but price was already hot, so high but not oversized.
- Korea side was the highest variance, so smallest.
- Korea Under 2.5 could be larger than Korea win when the total signal is clearer than the side.
- Canada sat between Mexico and Korea.

## Live execution checklist

- Use the approved Hong Kong execution server when the user says regional access matters.
- Re-fetch valid current `clobTokenId` from Polymarket Gamma data; do not reuse stale IDs from uploaded scripts.
- Verify orderbook before live order.
- Report matched status, order ID, tx hash, and balance movement.
- If the user's uploaded script has stale token IDs or old SDK order-version issues, preserve it for reference but place orders via the verified current SDK/helper after checking the market.
