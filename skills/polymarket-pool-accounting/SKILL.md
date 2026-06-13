---
name: polymarket-pool-accounting
description: Polymarket 多人资金池下注记账与赛后结算：按参与人余额占比分摊 account1/account2 每场、每个具体市场的下注成本，赛后按实际盈亏更新每人余额。
tags: [polymarket, betting, accounting, pool, pnl, settlement]
version: 1.0.0
author: Hermes Agent
related_skills: [world-cup-polymarket-orchestrator]
---

# Polymarket 多人资金池下注记账与结算

## 何时使用

当主人要求：

- “统计每个人余额和池子占比”
- “把 account1 当前下注按每个人 × 每场 × 具体市场细分”
- “每场比赛结束后，算盈亏和剩余余额”
- “某场下注成本/利润按粟粟、ray、Steve、小韩等分配”
- “给出每个人每场下了多少 USD 的 betting”

必须使用本 skill。

## 核心口径

### 1. 资金池余额

每个人余额来自本金 + 已实现利润分配 - 已实现亏损分配。若用户说“某人转让 X USD 给新子账户/另一账号”，这是资金池内部转让：池子总额不变，转出人余额减少，转入人/新子账户余额增加，并立即重新计算后续下注占比。**历史已下注市场仍按下注时快照占比分摊，不能用转让后的新占比重算历史仓位。**修改账本前先备份原文件，并在账本顶部记录转让说明和新占比。

示例：

```text
粟粟：781.3649
ray：327.1824
Steve：6754.3649
小韩：727.0000
假装在伦敦：727.0000
石桥：624.0000
```

资金池总额：

```text
pool_total = sum(person_balance)
```

每个人占比：

```text
person_pct = person_balance / pool_total
```

### 2. 按具体市场分摊下注

不能只按比赛粗分，必须拆到独立市场/token：

```text
USA vs Paraguay / USA 胜 Yes
USA vs Paraguay / USA Under 2.5
Canada vs Bosnia / Canada 胜 Yes
Canada vs Bosnia / Canada Under 2.5
Brazil vs Morocco / Brazil 胜 Yes
Brazil vs Morocco / Brazil Under 2.5
Haiti vs Scotland / Scotland 胜 Yes
Haiti vs Scotland / Scotland -1.5
Haiti vs Scotland / Over 2.5
```

每个人在某个市场的 betting：

```text
person_market_bet = market_cost * person_pct
```

按比赛小计：

```text
person_match_bet = sum(person_market_bet for markets in same match)
```

总分摊校验：

```text
sum(person_market_bet over all persons) ≈ market_cost
sum(person_all_bets) ≈ account_total_open_cost
```

## 示例解释：Haiti vs Scotland 三个市场

若石桥池子占比为 6.2771%，Haiti vs Scotland 当前三笔市场成本为：

```text
Scotland 胜 Yes：169.9977 USDC
Scotland -1.5：49.9967 USDC
Over 2.5：29.9982 USDC
```

石桥分摊：

```text
Scotland 胜 Yes = 169.9977 * 6.2771% = 10.6709 USD
Scotland -1.5 = 49.9967 * 6.2771% = 3.1383 USD
Over 2.5 = 29.9982 * 6.2771% = 1.8830 USD
```

含义：这是同一场比赛里的三个独立 Polymarket 市场，不能把“份额/股数”相加，只能把 USD 成本按金额合计。

## Account value and participant-profit reconciliation

When the user asks for account-level balances/profit, do not rely only on the normal `positions` list. Some combo/parlay/conditional markets may appear in activity history but not in ordinary positions output.

Required checks:

1. Read the pool ledger for participant principal/balance snapshots and transfer history.
2. Query current cash/balance from the Polymarket account.
3. Query open positions and sum `currentValue` / `cashPnl`.
4. Also inspect recent activity for combo/parlay titles such as `A AND B`, redeemed/closed markets, and manual sells. If a user-provided participant profit example implies a higher account net value than `cash + positions`, treat this as a reconciliation signal and locate the missing asset/value rather than reporting the lower number.
5. Distinguish three different profit views:
   - **Current floating PnL by pool share**: current net value minus current pool basis, split by current or historical share as appropriate.
   - **Profit vs original principal**: current participant balance minus that participant's earliest principal basis. This includes earlier realized profits already embedded in balances.
   - **Reset-to-principal after payout**: when the user says profits were paid separately, remove all profit from pool balances and keep only principal; future bets use the new principal-only percentages.

Original-principal examples from this pool family:

```text
粟粟: earliest principal 727; profit = current balance - 727.
ray: earliest principal 300.
Steve: if he transferred subaccounts out, adjust his current personal principal basis by subtracting transfers (e.g. 6000 - 739 - 1000 = 4261) while optionally showing a combined Steve-family view.
```

When resetting after profit payout, write an explicit reset note and recompute percentages from the principal-only total. Mark older market-level allocations as historical snapshots so future agents do not use stale percentages for new bets. If the user later provides a new total pool basis (e.g. “从现在开始置零，本金改为10264…超出来的部分划归Steve”), treat the latest reset as authoritative, recompute all percentages from that total, and append a new ledger note rather than trying to reconcile back to the prior reset.

If the user then says “从现在开始置零，本金改为 X，所有人返回最原始本金，超出来的部分划归 Steve”, treat `X` as the new account1 pool basis for future allocations. Hold every non-Steve participant at original principal, compute `Steve = X - sum(non_Steve_original_principals)`, and recompute all percentages from `X`. Save this as a hard cutover point; future bets use the new reset percentages.

If the user later moves principal between participants (for example “笑笑本金增加 295，从 Steve 扣除；松竹梅本金增加 739，从 Steve 扣除”), do **not** treat it as PnL. Keep the pool total fixed, debit Steve, credit the named participants, recompute percentages immediately, and regenerate any “当前下注按人/每场/每注分割” reports using the newest percentages. Older allocation files become historical snapshots.

## 赛后结算流程

每场比赛结束后：

1. 查询该场所有相关市场在 account1/account2 的最终状态：resolved / redeemable / realized PnL。
2. 对每个市场计算：

```text
market_profit = market_payout_or_sell_value - market_cost
```

3. 按赛前该市场分摊比例分配盈亏：

```text
person_market_pnl = market_profit * person_pct_at_bet_time
```

4. 更新每人余额：

```text
person_new_balance = person_old_balance + sum(person_market_pnl for resolved markets)
```

5. 重新计算全池：

```text
new_pool_total = sum(person_new_balance)
new_person_pct = person_new_balance / new_pool_total
```

6. 输出：

```text
每场总成本 / 总回收 / 总盈亏
每个市场成本 / 回收 / 盈亏
每个人分摊成本 / 盈亏 / 新余额 / 新占比
```

## 输出模板

```text
资金池总额：9940.9122 USD
当前下注总成本：1999.9764 USDC

参与人：
- 粟粟：余额 781.3649，占比 7.8601%
...

具体市场分摊：
### 石桥
- Haiti vs Scotland / Scotland 胜 Yes：10.6709 USD
- Haiti vs Scotland / Scotland -1.5：3.1383 USD
- Haiti vs Scotland / Over 2.5：1.8830 USD
- Haiti vs Scotland 小计：15.6922 USD

赛后：
- 本场总盈亏：...
- 石桥本场盈亏：...
- 石桥新余额：...
```

## 文件保存规范

保存到：

```text
/root/.hermes/private_tasks/coco/polymarket_betting_participants/
```

推荐文件：

```text
current_pool_market_level_allocation_YYYYMMDD.md
current_pool_market_level_allocation_YYYYMMDD.csv
match_settlement_<match_slug>_YYYYMMDD.md
match_settlement_<match_slug>_YYYYMMDD.csv
```

## References

- `references/account1-profit-reset-principal-20260613.md` — worked example of reconciling missing combo/parlay value, calculating profit vs earliest principal, and resetting the pool after profits were paid out.
- `references/account1-reset-10264-and-current-bet-allocation-20260613.md` — latest account1 reset-to-10264 principal basis, Steve excess allocation, add-50% Switzerland/Brazil orders, and per-person by-match allocation snapshot.
- `references/reset-zero-portfolio-value-excess-to-steve-20260613.md` — worked example of aligning account1 to a Polymarket screenshot `Total Portfolio Value`, resetting future allocations to zero, and assigning excess over original principals to Steve.
- `references/account1-principal-transfer-and-current-bet-reallocation-20260613.md` — worked example for post-reset internal principal transfers (e.g. credit 笑笑/松竹梅 from Steve), then re-splitting current open bets by match and by market using the latest percentages.

## 常见坑点

❌ 只按比赛粗分，不拆具体市场。  
✅ 必须拆到 moneyline、spread、total 等独立市场。

❌ 把不同市场的 shares/size 相加。  
✅ 不同 token 的 shares 不能相加，只能汇总 USD 成本。

❌ 只用普通 `positions` 接口算 account NAV/PnL。  
✅ 还要检查 activity / portfolio 中的 combo、parlay、conditional 市场（如 `Brazil win AND Switzerland win`）；这类仓位可能不出现在普通 positions 列表，但会影响当前净值和每人分摊盈亏。

❌ 赛后用最新占比分配历史下注盈亏。  
✅ 应用下注时的 `person_pct_at_bet_time` 分配该笔市场盈亏；若中途有人入金/让出本金，要保留快照。内部转让/新增子账户不回溯分配历史盈亏，除非主人明确指定。

❌ 忽略 account2。  
✅ 多账户时先按账户归集，再按具体市场汇总成本和盈亏，最后按资金池占比分配。

## 验证

- 每个市场分摊金额合计应等于该市场成本，允许 0.01 四舍五入差异。
- 每个人所有市场分摊合计应等于 `account_total_open_cost * person_pct`。
- 所有人分摊合计应等于 account 当前总下注成本。
