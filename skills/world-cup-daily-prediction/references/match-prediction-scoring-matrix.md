# Match Prediction Scoring Matrix v2

Use before assigning A/B/C grades in `world-cup-daily-prediction`.

## Required order

1. Verify fixture and lineup/injury facts.
2. Estimate model probability `q` before reading the market.
3. Read executable market buy price `p` from best ask/sell-one price with timestamp.
4. Compute edge: `q - p`.
5. Only mark executable if edge >= 5 percentage points.

## 25-point matrix

Score each item from 0-5:

- Fundamentals: Elo/ranking/xG/recent performance.
- Lineup and injuries: confirmed starters, position-weighted injury impact.
- Motivation and schedule: group incentives, rest, travel, rotation.
- Market price: edge vs best ask, liquidity, spread.
- Information certainty: source quality, lineup confirmation, data freshness.

## Grade mapping

- A: high matrix score, data complete, edge >= 5pp, strong counterargument addressed.
- B: edge >= 5pp but one or more uncertainty factors remain; size must be reduced by risk multipliers.
- C: edge < 5pp, data stale/incomplete, slug mismatch, or thesis too fragile. Output `无 edge / 不下`.

## Counterargument requirement

For every A/B recommendation, write:

- strongest counterargument;
- exact downgrade trigger;
- pass price where edge disappears;
- lineup player/position that changes the thesis.

## Calibration fields

After the match, append to the shared ledger:

```text
date / match / market / q / p / closing_price / result / CLV / notes
```

Rolling 50 bets feed into the Brier/CLV kill-switch in `sports-betting-risk-management`.
