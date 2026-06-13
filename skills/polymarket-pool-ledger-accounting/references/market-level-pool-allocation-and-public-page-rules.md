# Market-level pool allocation and public page rules

## Session-derived durable rules

Use this for Polymarket/World Cup participant accounting where multiple people share one account1 pool.

## Current accounting pattern

- Track each participant's balance.
- When Steve transfers/assigns funds to another participant, reduce Steve's balance and add the new participant balance, then recompute all percentages.
- Pool percentage = participant balance / total pool balance.
- Account2 is excluded from the shared pool by default; account2 can be used for separate live tests/trades but does not affect account1 pool accounting unless explicitly included.
- Allocation must be at market level, not only match level.

## Required allocation granularity

For each account1 position, allocate by participant:

```text
participant_market_bet = market_initial_value × participant_pool_pct
```

Examples of separate markets that must remain separate:

- `USA vs Paraguay / USA 胜 Yes`
- `USA vs Paraguay / USA Under 2.5`
- `Haiti vs Scotland / Scotland 胜 Yes`
- `Haiti vs Scotland / Scotland -1.5`
- `Haiti vs Scotland / Over 2.5`

Do not combine shares/size across markets. Only money can be summed.

## Haiti vs Scotland term explanations

- `Scotland 胜 Yes`: Scotland must win the match.
- `Scotland -1.5`: Scotland must win by at least 2 goals. 2-0 and 3-1 win; 1-0, 2-1, draw, or loss lose.
- `Over 2.5`: total goals must be at least 3. 2-1 and 3-0 win; 1-0, 1-1, and 2-0 lose.

## Public webpage rule

For shareable public pages, show only betting recommendations and term explanations. Exclude:

- internal account names (`account1`, `account2`)
- holdings/positions
- participant names
- balances and pool percentages
- internal pool/accounting wording

Use proportional display such as `每 100 单位` so recipients can scale amounts.

## After-match settlement

After each match ends:

1. Query market outcomes/positions/activity for all related account1 markets.
2. Calculate per-market realized PnL or settlement value.
3. Apply any explicit special allocation first; otherwise distribute PnL by current pool percentage.
4. Update each participant's balance.
5. Output per participant: stake, return, PnL, new balance, new pool percentage.
