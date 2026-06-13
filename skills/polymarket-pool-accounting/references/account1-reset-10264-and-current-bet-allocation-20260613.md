# Account1 reset-to-10264 and current bet allocation pattern (2026-06-13)

## Reset instruction

The user changed the pool accounting basis after reviewing Polymarket UI total portfolio value:

```text
从现在开始置零，本金改为10264，所有人都返回到最原始本金。超出来的部分划归steve账户。
```

This supersedes the earlier principal-only reset of 9548. The new account1 pool basis is:

```text
pool_total = 10264.0000
粟粟 = 727
ray = 300
Steve = 4977
小韩 = 727
假装在伦敦 = 727
石桥 = 624
笑笑 = 443
松竹梅 = 739
zeng hui = 1000
```

Reasoning:

```text
non-Steve original principal = 727 + 300 + 727 + 727 + 624 + 443 + 739 + 1000 = 5287
Steve = 10264 - 5287 = 4977
```

Future account1 bets and PnL after this reset use these percentages:

```text
粟粟 7.0830%
ray 2.9228%
Steve 48.4899%
小韩 7.0830%
假装在伦敦 7.0830%
石桥 6.0795%
笑笑 4.3161%
松竹梅 7.1999%
zeng hui 9.7428%
```

## Allocation after add-50% orders

After the reset, the user asked to add 50% to account1 Switzerland and Brazil exposure. Executed account1 orders:

```text
Switzerland win Yes: +792 @ 0.82 = 649.44 USDC
Switzerland -1.5: +593 @ 0.59 = 349.87 USDC
Brazil win Yes: +826 @ 0.59 = 487.34 USDC
```

Brazil Under 2.5 was not increased because it had been explicitly downgraded to a protection/hedge position after injury reassessment.

Account1 key positions after the add:

```text
Switzerland win Yes: size 2396, initialValue 1940.76
Switzerland -1.5: size 1800, initialValue 1044.00
Brazil win Yes: size 2110, initialValue 1232.0163
Brazil Under 2.5: size 412.3399, initialValue 224.9001
```

Current match-level cost allocation snapshot:

```text
Swiss match = 1940.76 + 1044.00 = 2984.7600
Brazil match = 1232.0163 + 224.9001 = 1456.9164
Scotland match = 769.7567 + 199.7568 + 280.1577 = 1249.6712
Swiss+Brazil combo = 101.5039
listed total = 5792.8515
```

Per-person by-match allocation using the 10264 reset basis:

```text
粟粟: Swiss 211.4108, Brazil 103.1935, Scotland 88.5143, combo 7.1895, total 410.3082
ray: Swiss 87.2397, Brazil 42.5833, Scotland 36.5259, combo 2.9668, total 169.3156
Steve: Swiss 1447.3062, Brazil 706.4568, Scotland 605.9639, combo 49.2191, total 2808.9460
小韩: Swiss 211.4108, Brazil 103.1935, Scotland 88.5143, combo 7.1895, total 410.3082
假装在伦敦: Swiss 211.4108, Brazil 103.1935, Scotland 88.5143, combo 7.1895, total 410.3082
石桥: Swiss 181.4585, Brazil 88.5732, Scotland 75.9738, combo 6.1709, total 352.1765
笑笑: Swiss 128.8239, Brazil 62.8813, Scotland 53.9365, combo 4.3810, total 250.0227
松竹梅: Swiss 214.9004, Brazil 104.8968, Scotland 89.9754, combo 7.3082, total 417.0808
zeng hui: Swiss 290.7989, Brazil 141.9443, Scotland 121.7528, combo 9.8893, total 564.3854
```

## Files produced in session

```text
/root/.hermes/private_tasks/coco/polymarket_betting_participants/reset_zero_principal_10264_excess_to_steve_20260613.md
/root/.hermes/private_tasks/coco/polymarket_betting_participants/account1_add_50pct_swiss_brazil_20260613.md
/root/.hermes/private_tasks/coco/polymarket_betting_participants/account1_current_bets_by_person_match_allocation_20260613.md
```

## Durable workflow lessons

- If the user says profits are paid out and then changes the pool total again, treat the latest pool-total instruction as authoritative and recompute percentages from scratch.
- Do not reuse historical allocation rows after a reset; mark old rows historical and use the new reset percentages for future bets.
- For “按目前下注来分割每个人的每场比例”, aggregate by match in USD cost, not by shares. Keep combo/parlay markets separate unless user explicitly asks to split them into legs.
