---
name: polymarket-pool-ledger-accounting
description: Polymarket 多人资金池记账与下注分摊：按参与人余额/池子占比，把 account1/account2 每场每个具体市场下注成本、赛后盈亏和剩余余额分摊到个人。
tags: [polymarket, ledger, betting, pool-accounting, profit-allocation, finance]
version: 1.0.0
author: Hermes Agent
related_skills: [world-cup-polymarket-orchestrator, sports-betting-risk-management]
---

# Polymarket 多人资金池下注记账与分摊

## 何时使用

当主人要求：

- “记录谁参加了下注 / 每个人多少钱”
- “按池子占比算每个人余额”
- “把 account1 当前下注按每个人 × 每场 × 具体市场细分”
- “比赛结束后算每个人盈亏和剩余余额”
- “某场的利润按某几个人固定比例分配”

使用本 skill。

## 当前基准账户与参与人示例

截至 2026-06-13 的已确认池子口径：

```text
粟粟：781.3649 USD
ray：327.1824 USD
Steve：5572.3649 USD
小韩：727.0000 USD
笑笑：443.0000 USD
假装在伦敦：727.0000 USD
石桥：624.0000 USD
松竹梅：739.0000 USD
总额：9940.9122 USD（约 9941）
总额：9940.9122 USD（约 9941）

口径：当前资金池统计只按 account1 持仓/下注计算；account2 的测试单或单独账户下注不进入这个池子总账，除非主人明确说 account2 也纳入整体统计。
最新账本默认读取：`/root/.hermes/private_tasks/coco/polymarket_betting_participants/latest_account1_pool.md`
```

背景修正：

```text
Steve 原本金改为 7427 USD
Steve 让出 727 USD 给小韩
第一场 Mexico vs South Africa 利润 135.9122 USDC
利润分配：粟粟 40%，ray 20%，Steve 40%
Steve 后续让出 443 USD 给笑笑
Steve 后续下划 739 USD 给松竹梅参与世界杯下注
```

因此余额最新口径：

```text
粟粟 = 727 + 135.9122*40% = 781.3649
ray = 300 + 135.9122*20% = 327.1824
Steve = 7427 - 727 - 443 - 739 + 135.9122*40% = 5572.3649
小韩 = 727
笑笑 = 443
假装在伦敦 = 727
石桥 = 624
松竹梅 = 739
```

其中：Steve 先让出 727 给小韩；后续又让出 443 给笑笑；2026-06-13 又从 Steve 下划 739 USD 给松竹梅参与 account1 世界杯下注池。
其中：Steve 先让出 727 给小韩，后续让出 443 给笑笑，又下划 739 给松竹梅。

## 核心公式

### 1. 池子占比

```text
个人占比 = 个人当前余额 / 池子账面总额
```

### 2. 具体市场下注分摊

每个市场单独算，不能只按比赛总额模糊分摊：

```text
个人在某市场分摊下注 = 该市场总成本 × 个人占比
```

示例：石桥余额 624，总池子 9940.9122，占比 6.2770899435%。

Haiti vs Scotland 三个市场：

```text
Scotland 胜 Yes：169.9977 × 6.2770899435% = 10.6709 USD
Scotland -1.5：49.9967 × 6.2770899435% = 3.1383 USD
Over 2.5：29.9982 × 6.2770899435% = 1.8830 USD
```

### 3. 比赛小计

```text
个人某场总下注 = sum(个人在该场所有具体市场分摊下注)
```

例如石桥 Haiti vs Scotland 小计：

```text
10.6709 + 3.1383 + 1.8830 = 15.6922 USD
```

### 4. 赛后盈亏分摊

赛后先按真实结算结果算每个市场 PnL：

```text
市场盈亏 = 结算/平仓回收金额 - 市场成本
```

再按个人占比分摊：

```text
个人市场盈亏 = 市场盈亏 × 个人占比
个人新余额 = 个人旧余额 + sum(个人已结算市场盈亏)
```

若某场存在特殊利润分配（例如 Mexico 第一场利润粟粟40%、ray20%、Steve40%），优先使用主人指定分配比例，不按池子占比分摊。

## 数据来源优先级

1. Polymarket `positions` 当前持仓：`initialValue/currentValue/cashPnl/curPrice`。
2. Polymarket `activity` 成交记录：用于核对 buy/sell、成交价、交易哈希。
3. CLOB order response：用于本次新增下单回执。
4. 本地资金池 ledger 文件：用于参与人本金、特殊利润分配、转让记录。

## 标准输出结构

### A. 参与人余额与占比

```text
- 粟粟：余额 X USD，占比 Y%
- ray：余额 X USD，占比 Y%
...
```

### B. account 当前下注总额

```text
account1 未平仓下注成本：1999.9764 USDC
池子账面总额：9940.9122 USD
```

### C. 按人 × 每场 × 具体市场

必须列到具体市场层级，例如：

```text
石桥
- USA vs Paraguay / USA 胜 Yes：37.6625 USD
- USA vs Paraguay / USA Under 2.5：25.1081 USD
- Haiti vs Scotland / Scotland 胜 Yes：10.6709 USD
- Haiti vs Scotland / Scotland -1.5：3.1383 USD
- Haiti vs Scotland / Over 2.5：1.8830 USD
- 合计分摊下注：125.5403 USD
```

### D. 赛后结算

每场结束后输出：

```text
比赛：A vs B
市场1：成本 / 回收 / 盈亏
市场2：成本 / 回收 / 盈亏
本场总盈亏：...

个人分摊：
- 粟粟：本场盈亏 ...，新余额 ...
- ray：本场盈亏 ...，新余额 ...
...
```

## 文件保存规范

默认保存到：

```text
/root/.hermes/private_tasks/coco/polymarket_betting_participants/
```

推荐文件：

```text
current_pool_market_level_allocation_YYYYMMDD.md
current_pool_market_level_allocation_YYYYMMDD.csv
match_settlement_<match_slug>_YYYYMMDD.md
match_settlement_<match_slug>_YYYYMMDD.csv
latest.md
```

当主人说“做个网页/方便发给别人看/展示出来”时，先判断用途：

### 内部核对网页

如果主人明确要求内部核对/给自己看，生成单文件 HTML 到：

```text
/root/.hermes/media_cache/polymarket_pool_account1_allocation_<date>.html
```

可包含：account1/account2 口径、每个人余额与占比、按人 × 具体市场分摊。

### 外部公开网页

如果主人说“方便发给别人看/外部可以访问/不需要真实展示持仓和朋友金额”，生成公开版网页到：

```text
/root/.hermes/media_cache/polymarket_public_betting_page/index.html
```

公开版只展示：

- 每场比赛建议如何下注；
- 每 100 单位的比例，方便缩放；
- 术语解释（胜 Yes、-1.5、Over/Under 2.5）；
- 风险提醒。

公开版禁止出现：账户名、持仓、朋友姓名、个人金额、池子占比、内部资金池信息，且不要写“本页不展示账户/朋友金额”这类反向暴露语句。详见 `references/market-level-pool-allocation-and-public-page-rules.md`。


## 关联参考

- `references/account1-pool-ledger-update-pitfalls-20260613.md`：新增 account1 实盘订单后更新资金池分摊表时使用；强调先备份、按参与人 line-by-line 插入、避免跨 section 正则误改、account2 单独记录不并入池子。

## 校验清单

输出前必须检查：

1. 个人余额合计是否等于池子账面总额。
2. 个人占比合计是否约等于 100%。
3. 每个市场的个人分摊合计是否约等于该市场总成本。
4. 每个人所有市场分摊合计是否等于其占比 × 总下注成本。
5. 不同市场的 shares/size 不能相加；只能合计 USDC 金额。
6. 赛后结算时，特殊利润分配优先于普通池子占比分配。

## 主人口令/简称解释

- “account1 里面从 A 下划 X USD 给 B 下注世界杯”：按 account1 资金池内部参与人转让/划拨处理，A 余额减 X，B 余额加 X；池子总额不变；立即重算所有参与人占比，并按现有 account1 每个具体市场成本重新分摊。除非主人另说“实际下单/买入/执行”，这只是资金池记账，不代表新增真实交易。
- “给一个某某”通常表示新增参与人或给既有参与人增加余额；若名称未出现过，新增参与人并在备注写明来源。
- 更新后必须同步最新指针文件（如 `latest_account1_pool.md` / `latest.md`）并输出校验：余额合计、占比合计、account1 当前下注总成本。

## 常见坑点

❌ 把资金池 ledger / 赛前分摊表当成真实 Polymarket 成交。  
✅ 主人要求“真实盈亏/读 account 持仓和历史”时，必须先查 data-api `positions` + `activity/trades`，用真实 proxyWallet、market slug、tx hash 复核；若 ledger 里有某场但 account activity 无该场，必须明确说“这是分摊/计划口径，不是 account 可核验真实成交”。

❌ 把“下划/划给某人”误解为立即新增下单。  
✅ 先作为资金池参与人余额调整；真实 Polymarket 交易需要主人明确市场、方向、金额并再次确认。

❌ 只把 USA vs Paraguay 合成一行，不拆 USA 胜和 Under 2.5。  
✅ 必须拆到具体市场：`USA 胜 Yes`、`USA Under 2.5`。

❌ 把不同 market 的“股数/份额”相加。  
✅ 只合计金额 USDC；股数/size 必须按市场单列。

❌ 用贪婪/跨 section 正则一次性修改整份 Markdown 分摊表，容易把新增市场只插到最后一个人、并错误改大最后一个人的合计。  
✅ 更新 `latest_account1_pool.md` 前先备份；优先逐行扫描 `### 参与人` section，在每个 section 的 `- 合计分摊下注` 前插入新增市场分摊，并按该人余额/池子总额重算该人的合计。写完必须校验：所有个人合计之和≈市场成本合计；新增市场各人的分摊合计≈新增真实成交成本；若异常，立即从备份恢复重做。

❌ 忘记 Steve 让出 727 给小韩、443 给笑笑、739 给松竹梅。  
✅ 当前 Steve 余额按 5572.3649，小韩 727，笑笑 443，松竹梅 739 计算。

❌ 赛后所有利润都按池子占比。  
✅ 若主人指定某场利润分配比例，按指定比例覆盖默认池子占比。
