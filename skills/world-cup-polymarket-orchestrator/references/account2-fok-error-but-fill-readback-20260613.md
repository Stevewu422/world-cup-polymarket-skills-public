# Account2 FOK error but actual fill readback pattern (2026-06-13)

## What happened

During an account2 “increase 50% position” run, the first `Switzerland win` FOK order returned an API exception:

```text
order couldn't be fully filled. FOK orders are fully filled or killed
```

However, a subsequent readback of account2 activity and positions showed that the `Switzerland win` order had actually entered the account:

```text
Switzerland win size increased from 802 to 1198
recent activity showed BUY size 396 @ 0.82
```

The safe response was **not** to immediately retry the same order. The correct response was to query activity/positions first and only execute the remaining markets.

## Durable rule

For Polymarket live trading, especially account2 custom-script execution:

1. If `post_order` / `create_and_post_order` raises an exception after reaching the CLOB API, do not assume no fill.
2. Immediately read back:
   - `data-api.polymarket.com/activity?user=<funder>&limit=...`
   - `data-api.polymarket.com/positions?user=<funder>&limit=...`
3. Compare pre/post size for the target token.
4. Only retry if both activity and positions confirm no fill.
5. In the user report, say “script returned error, but readback confirmed fill” without exposing credentials.

## Why it matters

Blind retry after an ambiguous FOK/API exception can double the intended position. This is especially dangerous during “increase X%” instructions.

## User-facing wording

```text
第一笔脚本返回 FOK 报错，但我重新回读 activity 和 positions，确认这笔已经实际成交并进入持仓，所以没有重复下同一笔。
```
