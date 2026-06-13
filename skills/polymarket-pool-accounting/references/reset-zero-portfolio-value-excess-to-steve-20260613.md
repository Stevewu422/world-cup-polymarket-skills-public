# account1 reset-to-zero with platform portfolio value and excess-to-Steve pattern

Session date: 2026-06-13

## Trigger

User reviewed a Polymarket portfolio screenshot and corrected the pool accounting:

- Treat Polymarket `Total Portfolio Value` as the current account1 pool value when the user asks to align the pool with the platform UI.
- From a chosen reset point, set everyone back to original principal.
- Any amount above the sum of non-Steve original principals is assigned to Steve.
- From that moment onward, new bets and new PnL use the new reset percentages.

## Example from this session

Platform screenshot:

```text
Total Portfolio Value = 10264.99
Available to Trade = 5853.00
Profit/Loss = 344.05  # platform period P/L, not necessarily pool cumulative P/L
```

User then instructed:

```text
从现在开始置零，本金改为10264，所有人都返回到最原始本金。超出来的部分划归steve账户。
```

Applied reset with total principal rounded to `10264.0000`:

```text
Non-Steve original principals:
粟粟 727
ray 300
小韩 727
假装在伦敦 727
石桥 624
笑笑 443
松竹梅 739
zeng hui 1000
sum = 5287

Steve new principal = 10264 - 5287 = 4977
```

New percentages:

```text
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

## Required workflow for future agents

1. **Clarify the accounting view from the user's wording.**
   - `当前总池子/截图总值` → use platform Total Portfolio Value.
   - `利润已支付，只保留本金` → reset to principal-only.
   - `本金改为 X，超出归 Steve` → set pool total to X; hold non-Steve principals fixed; compute Steve as residual.
2. **Do not keep old profit-embedded balances after reset.** Once reset-to-zero is confirmed, future new bets use reset percentages only.
3. **Keep historical allocation sections explicitly labeled as historical snapshots** so future agents do not reuse stale percentages.
4. **Record a reset note in `latest_account1_pool.md` and save a session-specific snapshot file.**
5. **Verify sums:** balances sum to target pool total and percentages sum to 100.0000% within rounding tolerance.

## Pitfall

Do not mix these three concepts in one answer:

- platform current total value,
- original-principal profit view,
- reset principal basis for future betting.

When the user says “从现在开始置零”, that is a hard cutover point for future allocations.