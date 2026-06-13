# Account1 principal-transfer and current-bet reallocation pattern (2026-06-13)

## Trigger

Use when the user changes account1 participant principals after a reset and then asks to re-split current open bets, e.g.:

```text
笑笑的本金增加 295 美金，从 Steve 那边扣除。
松竹梅本金增加 739，从 Steve 那边扣除。
按目前下注来重新分割每个人的每场比例。
```

## Durable rules

1. Treat this as an internal principal transfer, not a profit/loss event.
2. Pool total stays fixed unless the user explicitly changes it.
3. Debit Steve and credit the named participants immediately.
4. Recompute percentages from the new principal balances before any new allocation.
5. When the user asks “按目前下注”, split **current open bet cost** by the latest principal percentages.
6. Separate by match first, then if requested split into each independent market/outcome.
7. Combo/parlay positions such as `Brazil win AND Switzerland win` should be listed as a separate bucket unless the user explicitly asks to force-allocate it into component matches.
8. Update the main ledger and save a dated allocation snapshot; mark older allocations as historical if the percentages changed.

## Worked example

After reset-to-10264, user changed principals:

```text
笑笑 +295 from Steve
松竹梅 +739 from Steve
Steve decrease = 1034
```

New principal pool remains 10264:

```text
粟粟 727 -> 7.0830%
ray 300 -> 2.9228%
Steve 3943 -> 38.4158%
小韩 727 -> 7.0830%
假装在伦敦 727 -> 7.0830%
石桥 624 -> 6.0795%
笑笑 738 -> 7.1902%
松竹梅 1478 -> 14.3998%
zeng hui 1000 -> 9.7428%
```

Current account1 open-bet cost snapshot used in this session:

```text
瑞士场 = 2984.7600  # 瑞士胜 + 瑞士 -1.5
巴西场 = 1456.9164  # 巴西胜 + Brazil Under 2.5
苏格兰场 = 1249.6712  # 苏格兰胜 + 苏格兰 -1.5 + Over 2.5
瑞士+巴西组合仓 = 101.5039
总下注成本 = 5792.8515
```

By-match allocation after the transfer:

```text
粟粟: 瑞士 211.4108, 巴西 103.1935, 苏格兰 88.5143, 组合 7.1895, 合计 410.3082
ray: 瑞士 87.2397, 巴西 42.5833, 苏格兰 36.5259, 组合 2.9668, 合计 169.3156
Steve: 瑞士 1146.6201, 巴西 559.6864, 苏格兰 480.0715, 组合 38.9936, 合计 2225.3715
小韩: 瑞士 211.4108, 巴西 103.1935, 苏格兰 88.5143, 组合 7.1895, 合计 410.3082
假装在伦敦: 瑞士 211.4108, 巴西 103.1935, 苏格兰 88.5143, 组合 7.1895, 合计 410.3082
石桥: 瑞士 181.4585, 巴西 88.5732, 苏格兰 75.9738, 组合 6.1709, 合计 352.1765
笑笑: 瑞士 214.6096, 巴西 104.7549, 苏格兰 89.8536, 组合 7.2983, 合计 416.5164
松竹梅: 瑞士 429.8008, 巴西 209.7937, 苏格兰 179.9507, 组合 14.6164, 合计 834.1616
zeng hui: 瑞士 290.7989, 巴西 141.9443, 苏格兰 121.7528, 组合 9.8893, 合计 564.3854
```

## Verification checklist

- Sum principals equals the pool basis (10264 in the worked example).
- Sum percentages ≈ 100% within rounding.
- For each match/market, sum participant allocations equals that match/market cost.
- Sum all participant totals equals the current open-bet cost snapshot.
- Report in Telegram as bullets, not markdown tables, because Telegram table syntax is poor.
