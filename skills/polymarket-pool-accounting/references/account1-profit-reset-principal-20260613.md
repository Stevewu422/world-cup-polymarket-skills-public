# Account1 profit reconciliation and reset-to-principal pattern (2026-06-13)

## What happened

During an account1 pool review, a first pass underreported participant profits because it used normal `positions` output only. The user corrected with an anchor example:

```text
粟粟 earliest principal = 727
粟粟 current balance = 805.1682
profit = 805.1682 - 727 = 78.1682
```

The missing amount came from a combo/parlay-style market (`Brazil win AND Switzerland win`) visible in activity but not represented in the normal positions list used for the first calculation.

## Correct workflow

1. Start from the pool ledger participant balances and transfer history.
2. Query cash and normal open positions.
3. Inspect recent activity for combo/parlay/conditional markets (`AND` titles, blank slug assets, or markets not in `positions`).
4. Cross-check the total account net value against at least one participant anchor if the user provides one.
5. Compute participant profit in the requested view:
   - current floating PnL by share, or
   - profit vs earliest principal, or
   - principal-only balances after profit payout.

## Original-principal profit example

Corrected balances after including the missing combo value and Steve transfer:

```text
粟粟: principal 727, current 805.1682, profit +78.1682
ray: principal 300, current 337.1496, profit +37.1496
Steve: principal basis 4261 after transfers, current 4742.1200, profit +481.1200
小韩: principal 727, current 749.1471, profit +22.1471
假装在伦敦: principal 727, current 749.1471, profit +22.1471
石桥: principal 624, current 643.0094, profit +19.0094
笑笑: principal 443, current 456.4954, profit +13.4954
松竹梅: principal 739, current 761.5127, profit +22.5127
zeng hui: new transfer 1000, current 1000, profit +0
```

## Reset after profit payout

When the user says all profit has been paid separately, reset the pool to principal-only balances and recompute future percentages from the principal total.

Historical intermediate reset (superseded later in the same session):

```text
principal total = 9548.0000
粟粟 727 -> 7.6142%
ray 300 -> 3.1420%
Steve 4261 -> 44.6271%
小韩 727 -> 7.6142%
假装在伦敦 727 -> 7.6142%
石桥 624 -> 6.5354%
笑笑 443 -> 4.6397%
松竹梅 739 -> 7.7398%
zeng hui 1000 -> 10.4734%
```

Latest user-confirmed reset from 2026-06-13 onward: the pool was reset to total principal `10264.0000`; everyone returned to earliest principal and all excess was assigned to Steve. Use this as the default current account1 pool basis unless the user changes it again:

```text
principal total = 10264.0000
粟粟 727 -> 7.0830%
ray 300 -> 2.9228%
Steve 4977 -> 48.4899%
小韩 727 -> 7.0830%
假装在伦敦 727 -> 7.0830%
石桥 624 -> 6.0795%
笑笑 443 -> 4.3161%
松竹梅 739 -> 7.1999%
zeng hui 1000 -> 9.7428%
```

Add an explicit ledger note that old market allocations are historical snapshots and new bets use the latest reset percentages. When the user asks to split current bets, split by both match and individual market/token (moneyline/spread/total/combo), not just by match.
