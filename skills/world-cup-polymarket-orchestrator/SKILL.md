---
name: world-cup-polymarket-orchestrator
description: 世界杯/足球 Polymarket 投研-预测-风控-执行总控：整合 research-source-monitoring、world-cup-daily-prediction、sports-betting-risk-management、world-cup-caiqiu-prediction，用于赛程/信息源监控、早盘/临场预测、比例仓位、香港服务器 CLOB 实盘执行与赛后复盘。
tags: [world-cup, football, soccer, polymarket, betting, prediction, risk-management, execution]
version: 1.0.0
author: Hermes Agent
related_skills: [research-source-monitoring, world-cup-daily-prediction, sports-betting-risk-management, world-cup-caiqiu-prediction, quant-trading-scripts, polymarket-pool-ledger-accounting]
---

# 世界杯 Polymarket 投研风控执行总控

## Session references

- `references/injury-adjusted-matchday-white-page-20260613.md` — injury-driven reforecast pattern for four World Cup matches, including downgrade discipline and publishing a verified white long-image page.
- `references/large-add-position-precision-and-market-selection-20260613.md` — large add-position execution notes: per-command max-order override, integer BUY size precision fix, and not adding to markets that were downgraded to protection/observation positions.
- `references/official-lineup-confirmation-window.md` — official XI timing discipline: start watch at T-70, treat leaked/predicted XI as PROVISIONAL, upgrade only after official/FIFA/team/Sofascore/FotMob confirmed XI around T-60, then re-estimate q and rerun edge gate.
- `references/v2-edge-gate-reforecast-hold-vs-add-20260614.md` — v2 reforecast pattern: separate HOLD from ADD, require `q - ask >= 5pp` for new exposure, and use ESPN/Gamma/CLOB probes when web search is unavailable.

## v2 执行护栏补丁（只加刹车，不优化执行机制）

# Orchestrator 护栏补丁 v1

> 本补丁**只往 `world-cup-polymarket-orchestrator` 新增护栏与警告**，
> 不改、不优化任何执行机制。
> **触碰范围**：执行前刹车门、数据完整性门、单一来源指针、风险/合规警告。
> **明确不触碰**：香港服务器出口路由、多账户协调逻辑、资金池分配记账、下单精度/吞吐。
> 这些只增加"在什么情况下不许下单"和"有哪些风险"，不会让上述任何能力更好用。

---

## 1. 执行前校准门（硬 gate，置于"实盘执行安全门"之前）

进入任何实盘执行前，必须先通过 `sports-betting-risk-management` 的校准检查：

- 读取滚动 50 注的 `model Brier` vs `市场基准 Brier`、以及 `平均 CLV`。
- **若 `model Brier ≥ 基准` 且 `平均 CLV ≤ 0` → 本轮一切实盘执行 `BLOCKED`**，
  只允许研究 / dry-run，不允许真金下单。
- 若模型仅边际优于基准（差距在噪声内）→ 所有 size `× 0.5`。

> 这是一道刹车：系统没被证明有 edge 时，不放行真金下单。
> 涉及他人资金时，这道门尤其重要——它是"系统其实没用却继续下注"的最后拦截。

---

## 2. 执行前数据完整性门（任一失败即 abort 该腿）

任何下单前必须全部通过：

- **FIXTURE**：对阵 / 日期 / 分组与官方源一致（否则 `FIXTURE_MISMATCH`）。
- **SLUG**：Polymarket slug 的队伍三字码与对阵匹配
  （防 `ger-kor` 类错链真金下单；不匹配 → `SLUG_MISMATCH`，撤下该链接）。
- **PRICE**：`clobTokenId` 取自当前 Gamma 数据、orderbook 有效、价格时间戳 ≤ 15 分钟。
- **SIZE**：精度合规（maker/taker 小数位）；不同 market 的份额不跨市场相加。

---

## 3. 仓位与停手：单一来源

- orchestrator **删除本地副本**：
  - "建议比例：A 1.2-1.5 份 / A- 1.0-1.2 份 / …"
  - "连续错 3 场停止"
- 全部改为调用 `sports-betting-risk-management` v2：
  edge 推导仓位（1/4 凯利）、回撤/置信区间停手、校准 Kill-switch。

> 三个文件各写一份阈值，迟早改岔。只留一个来源。

---

## 4. 风险与合规警告（请置于 orchestrator 文件顶部显著位置）

本系统涉及三类高风险、可能违规的操作。**技术能跑通 ≠ 合规**，正式运行前请由
当地有资质的法律/合规专业人士评估：

- **地区限制规避**：通过香港服务器并校验出口为 HK 来满足 Polymarket 访问，
  实质是绕开平台地理限制。这与 `world-cup-daily-prediction` 4.1 第 9 条
  "不要尝试规避地区限制"**直接冲突**，且可能违反平台条款，导致封号、冻结资金
  （连带损害资金池其他参与人）。
- **多账户控制**：一人控制多个账户通常违反平台"一人一户"条款；
  跨账户协调即便标注"避免自成交/操纵"，仍可能被认定为市场操纵。
- **多人资金池**：汇集他人资金代为下注并分配盈亏，在多个法域可能触及
  无牌资金管理 / 博彩经营的红线，具体取决于你和各参与人所在地。

**处理他人资金时**，任何突破风控上限的请求都必须二次确认并书面留痕；
风控规则对资金池与对自己一视同仁。

---

## 不在本补丁范围内（保持原样，亦不由本补丁优化）

- 香港服务器 / 出口国家校验 / HK guard 的实现细节
- 多账户认证、协调、汇总报告逻辑
- 资金池本金修正、利润分配、份额转让的记账逻辑
- CLOB 下单精度修正、FOK/重试、吞吐相关脚本

以上若需评估，请走合规/法律专业人士，而非把它当作纯技术问题。


## 何时使用

当主人要求：

- “世界杯/足球今天怎么下”
- “Polymarket 世界杯下单/预测/复盘”
- “04:00 早盘 / 16:00 临场”
- “按一份 10USDC，但要按比例下注”
- “列 Polymarket 市场、clobTokenId、大小球、胜负方向”
- “帮我从香港服务器实盘买入/补仓/调整仓位”

必须加载本总控 skill，并同时遵循下列子 skill 的职责：

- `research-source-monitoring`：信息源、赛程、盘口、市场监控。
- `world-cup-daily-prediction`：每日早盘/临场预测更新。
- `sports-betting-risk-management`：仓位比例、止损、风控纪律。
- `world-cup-caiqiu-prediction`：世界杯逐场执行卡、Polymarket 解析与执行口径。
- `quant-trading-scripts`：CLOB/香港服务器/脚本/组合/PnL 等 Polymarket 工具。

## 总流程

```text
信息源监控
  ↓
赛程/市场/赔率核验
  ↓
早盘或临场预测
  ↓
风险评级与比例仓位
  ↓
Polymarket 市场匹配与 clobTokenId 验证
  ↓
dry-run / live 执行安全检查
  ↓
香港服务器下单
  ↓
成交回报
  ↓
赛后复盘与下一轮权重修正
```

## 角色分工

### 1. 信息源雷达：research-source-monitoring

负责：

- 赛程、开球时间、时区核验；
- 伤停、首发、轮换、天气、场地；
- Polymarket event/market 页面发现；
- 价格/盘口变化、异常波动监控；
- 仅做只读查询时优先走该层。

### 2. 每日预测引擎：world-cup-daily-prediction

负责：

- 04:00 早盘：隔夜复盘 + 未来 24-36 小时初判；
- 16:00 临场：首发/伤停/盘口/Polymarket 价格复核；
- 对早盘判断执行“上调/维持/降级/取消”；
- 输出主方向、副方向、观望单。

### 3. 风控仓位引擎：sports-betting-risk-management

负责：

- 价值差门槛：用买入价/卖一价计算 `q-p`，低于 5pp 写“无 edge / 不下”；
- 先估模型概率 `q`，再读盘口 `p`，防止市价锚定；
- 仓位由 edge 推导：1/4 Kelly，信心/首发/流动性/相关性只能下修；
- 停手用回撤/置信区间，不再用“连错3场”硬停；
- 校准账本、Brier/CLV Kill-switch；
- 同方向跨场敞口上限；
- 禁止追单、倍投、马丁格尔。

### 4. 世界杯执行层：world-cup-caiqiu-prediction

负责：

- 逐场执行卡；
- 胜负、大小球、让球、观望；
- Gamma API `outcomes / outcomePrices / clobTokenIds` 解析；
- 大小球 Under/Over token 映射；
- 香港服务器实盘前检查和成交回报。

## 数据核验优先级

每次输出前必须尽量核验当前事实；不能凭旧记忆硬报：

1. 赛程/开球时间：赛事官网、FIFA、ESPN、BBC、Sofascore、FotMob。
2. 结果/积分/晋级形势：赛事官网、ESPN、BBC、Sofascore。
3. 阵容/伤停：球队官网、赛事官网、Sofascore/FotMob lineup、主流体育媒体。
4. 赔率/盘口/价格：Polymarket Gamma API、页面市场、赔率聚合源。
5. 天气/场地：只在显著影响比赛时纳入。

若实时数据不可得，必须写：`暂无可靠实时数据`，并改用条件阈值。

### 只读 API 探针优先路径

当常规网页搜索/页面抽取不足以稳定返回赛程、盘口或 More Markets 时，优先用 `references/live-fixture-market-api-probes.md` 的只读探针组合：

- 当前时间：`date -u` + `TZ=Asia/Shanghai date`。
- FIFA 赛程：`https://api.fifa.com/api/v3/calendar/matches?from=...&to=...&idCompetition=17&language=en`。
- ESPN 赛程/赔率：`https://site.api.espn.com/apis/site/v2/sports/soccer/fifa.world/scoreboard?dates=YYYYMMDD`；summary 可查 `boxscore/rosters/news/odds`，但若没有明确首发/伤停字段，必须写“首发/伤停待确认”。
- Polymarket 主事件：`https://gamma-api.polymarket.com/events?closed=false&limit=100&series_id=11433`。
- Polymarket 子市场/More Markets：`https://gamma-api.polymarket.com/events/keyset?tag_id=102232&series_id=11433&closed=false&order=startTime&ascending=true&limit=100&include_best_lines=true`。
- CLOB orderbook：`https://clob.polymarket.com/book?token_id=<clobTokenId>`，用 bids 最大价和 asks 最小价报告 best bid / ask。

## 输出卡片模板

```text
比赛：A vs B
时间：北京时间 YYYY-MM-DD HH:MM
Polymarket：URL / event slug

信息源状态：赛程已核验 / 首发待确认 / 伤停待确认 / 盘口已核验

预测：
- 主方向：...
- 副方向：...
- 不碰：...

评级：
- 主方向信心：A/B/C
- 价值等级：高/中/低
- 风险点：...

仓位：
- 目标份数：X 份（1份=10USDC 时约 X*10USDC）
- 现有持仓：Y 份
- 本次补单：max(0, X-Y) 份
- 仓位理由：为什么比其他场大/小

执行阈值：
- 可买价格：<= ...
- 降级条件：...
- 取消条件：...

Polymarket token：
- market title：...
- outcome：Yes/No
- clobTokenId：...
- orderbook：best bid / ask

风控：
- 单场总风险：不超过 ... 份
- 单日总投入：不超过 ... 份
- 停手机制：...
```

## 比例下注规则

仓位与停手规则统一引用 `sports-betting-risk-management` v2，本 orchestrator 不再维护独立份数表，避免多文件阈值改岔。

执行摘要：

- 先独立估 `q`，再读取 Polymarket / 赔率市场买入价 `p`。
- 只有 `q-p >= 5个百分点` 才可进入实盘候选；否则 `无 edge / 不下`。
- 仓位基数：`base = 0.25 * (q-p)/(1-p)`。
- 信心、首发、伤停、流动性、盘口时间戳、相关性、同方向敞口只能缩小仓位，不能放大。
- 旧 A/B/C 份数区间只作为 sanity 上限，不作为固定份数来源。
- 单日回撤 3% 停新增；峰值回撤 8% 全仓减半；50 注 Brier/CLV kill-switch 触发则停止真金下注。
- 用户明确说“买 X USDC / 再下 X USDC”时，仍需先过校准门、数据完整性门和合规风险提示；未过门只允许 dry-run。
- 报表口径：可汇总每场 USDC 金额敞口；但胜负、大小球、让球等不同市场的“份额/size”不能相加，必须按独立 market/outcome 分别列出。

## Polymarket 市场解析规则

### 获取有效 clobTokenId

优先从 Gamma API 当前 event/market 数据中取：

```text
markets[].outcomes
markets[].outcomePrices
markets[].clobTokenIds
```

映射：

```text
outcomes[0] <-> clobTokenIds[0]
outcomes[1] <-> clobTokenIds[1]
```

不能直接相信截图、旧脚本或用户历史 token；必须查 orderbook。

### 大小球 token 映射

对 totals 市场：

```text
line=2.5
outcomes[0] 通常是 Yes/Over
outcomes[1] 通常是 No/Under
```

买 Under 2.5 时，通常买 `line=2.5` 的 `No` token；仍需以当前 market 的 `outcomes` 实际顺序为准。

### World Cup games keyset

普通搜索漏掉 child / More Markets 时，优先用 keyset 类接口/页面数据查找世界杯 games 下的完整 event/market，再回到 Gamma market 数据验证。

## 校准门与数据完整性门（v2 硬 gate）

任何 live 实盘执行前必须先通过：

1. **校准门**：读取 rolling 50 bets 的 model Brier、market baseline Brier、average CLV；若 `model Brier >= baseline Brier` 且 `average CLV <= 0`，本轮实盘 `BLOCKED`，只允许 research/dry-run。
2. **数据完整性门**：fixture / date / group 交叉核验；Polymarket slug 队伍三字码匹配；market title、outcome、clobTokenId、orderbook 有效；价格时间戳 <= 15 分钟；SIZE 精度合规。任一失败即 abort 该腿。
3. **合规提示**：HK 路由、多账户、资金池分账/提成可能涉及平台条款、监管、税务、托管/受托责任，不应作为纯技术优化；规模化前需有资质专业人士评估。

## 实盘执行安全门

实盘下注是高风险动作。只有主人明确说“买入/下注/下单/补仓”，并且目标市场/方向/金额可确定时，才进入执行。

执行前必须：

1. 使用香港服务器 `<YOUR_HK_SERVER_IP>`，当前工作目录：`/opt/polymarket-worldcup`；
2. 验证公网出口国家/城市为 `HK / Hong Kong`；
3. 读取当前 Polymarket market，确认 active=true、closed=false；
4. 取最新 `clobTokenId`，不要复用旧 token；
5. 查 orderbook，确认不是 `invalid token id` / `No orderbook exists`；
6. 查 balance/allowance，确认余额和授权足够；
7. 计算本次补单份数和 notional，尊重 `POLYMARKET_MAX_ORDER_USD`；
8. 高于既定仓位上限时先提醒，不擅自加码；
9. 不回显 private key / API secret / passphrase。

成交后报告：

```text
match / market / outcome
price / size / notional
status: matched/open/failed
orderID
tx hash
before/after balance
```

## 早盘 / 临场 / 复盘模式

### 04:00 早盘

输出：

- 过去 24 小时结果复盘；
- 未来 24-36 小时赛程；
- 初版主方向/副方向；
- 初版比例仓位；
- 临场需复核变量。

### 16:00 临场

输出：

- 首发/伤停/盘口/Polymarket 价格变化；
- 对早盘每场判断：上调/维持/降级/取消；
- 最终执行卡；
- 若用户要求实盘，按香港服务器安全门执行。

### 赛后复盘

输出：

- 预测方向是否正确；
- 大小球是否正确；
- 偏差原因：阵容、红牌、早段进球、盘口误判、价格无价值；
- 下一轮修正：仓位上调/下调、模型权重调整。

- `references/account2-fok-error-but-fill-readback-20260613.md` — account2-specific pattern for ambiguous FOK/API errors where readback shows the order actually filled; always check activity/positions before retrying.
- `references/account-position-review-and-v2-trim-runbook.md` — use when reviewing account1/account2 current holdings and giving modification advice: query positions + activity, include combo/conditional markets, compare total exposure to v2 risk bands, and trim secondary/correlated markets before touching core side positions.

## 常见坑点

❌ 所有比赛机械下一份。  
✅ 按信心和赔率价值给比例，强的多，弱的少。

❌ 把同一比赛里的胜负市场份额和大小球市场份额相加，写成“本场合计 X 份”。  
✅ 不同 market 的份额不能合并；每场只能合计金额敞口，份额必须按独立 market/outcome 分别列。示例：`Canada 胜：103.96 份` 与 `Canada Under 2.5：85.92 份` 分开报，不得相加为 189.88 份。

❌ 把“总持仓补到 300 USDC”误解成“本轮新增买入 300 USDC”。  
✅ 用户说“这轮总共买入/再下/追加 N USDC”时，按本轮新增 notional 计算；若已有老仓，必须先区分 `已有持仓`、`本轮新增`、`最终总持仓`。

❌ 用用户上传脚本里的旧 token 直接 post_order。  
✅ 重新从 Gamma 当前数据取 `clobTokenId`，orderbook 验证后再执行。

❌ 按“目标金额 / 价格”算出很多小数位 size 后直接下大额 FOK/BUY，可能触发 Polymarket `invalid amounts, the market buy orders maker amount supports a max accuracy of 2 decimals, taker amount a max of 4 decimals`。  
✅ 下单前先做 dry-run；若报金额精度错误，把 `size` 调整为整数或最多 2 位小数，并确保 `price * size` 的 USDC maker amount 只有 2 位小数（例如 1300@0.81 可改为 size=1604、成本 1299.24；700@0.58 可改为 size=1207、成本 700.06）。重试后必须复核 order response、balance、positions 和 activity/trades。

❌ account2 FOK/API call raises an exception and you immediately retry the same leg.  
✅ Before retrying account2, query `positions` and recent `activity`; the order may have actually filled even if the client raised. If the intended size/price appears in activity or positions changed, mark that leg filled and only retry missing legs. See `references/account2-add50-execution-readback-20260613.md`.

❌ 从非香港服务器真钱下单。  
✅ 只从 `<YOUR_HK_SERVER_IP>` 香港服务器执行，并在脚本内做 HK guard。

❌ 把大小球 Under 2.5 买成 Over/Yes。  
✅ 核对 `outcomes` 顺序；通常 Under 是 totals line=2.5 的 No token。

❌ 亏了加倍追回。  
✅ 停手，不追单不倍投。

## 关联参考

更多细节见本 skill 的 `references/`：

- `references/orchestrator-workflow.md`
- `references/polymarket-execution-checklist.md`
- `references/proportional-staking-template.md`
- `references/polymarket-position-reporting-and-round-notional.md`：下注后报告口径；金额可按场合计，份额不能跨胜负/大小球市场相加；区分本轮新增与最终总持仓。
- `references/polymarket-position-close-runbook.md`：用户要求“平掉/close/flatten”已有 Polymarket 仓位时使用；包含卖出平仓、HK 执行、全场/半场/单队 O/U 区分，以及成交后用 positions + activity 双重复核的步骤。
- `references/polymarket-position-close-and-risk-override-runbook.md`：本次会话补充的实盘细节；覆盖 positions/activity 双源核验、平仓后 PnL 计算、order response 与 data-api `usdcSize` 差异、以及用户提出远超风控额度（如 1000 USDC 单场）时的主动降风险与二次确认流程。
- `references/large-stake-override-and-position-accounting.md`：用户明确要求突破常规仓位（如坚持下 1000、每场 250、补到总金额 500）时使用；包含一次风险提醒后按确认执行、目标总额补仓计算、order/activity 与 positions 两套口径分开报告、以及大额下单后的复核格式。
- `references/bankroll-capital-risk-and-large-order-guardrails.md`：按总资本（如 10,000 USDC）评估下注是否合理时使用；要求输出总敞口/单场/单市场占本金百分比、大额订单风险卡、15%-25% 总敞口停止新增规则，以及大额执行后的 positions/activity 复核。
- `references/polymarket-multi-account-control-plan.md`：需要同时管理多个主人授权的 Polymarket 账户时使用；包含账户配置、单账户/全账户风控、dry-run/live 执行、避免自成交/操纵/冲突方向、多账户汇总报告和故障处理。
- `references/account2-direct-script-execution-and-risk-override.md`：用户提供 account2 独立 Python 下单脚本并明确要求执行时使用；记录 `create_or_derive_api_key()` 可绕开 env 中无效 API 凭据、用户明确风险 override 后执行、以及用 positions/activity 复核的流程。
- `references/account2-add50-execution-readback-20260613.md`：account2 延续 account1 加仓口径执行瑞士/巴西 add-50% 时使用；重点记录“FOK/API异常后先读 positions+activity 再重试，避免重复成交”的坑。
- `references/pool-ledger-market-level-accounting-and-public-page.md`：多人资金池/竞猜池记账、account1 当前下注按“人 × 比赛 × 具体市场”分摊、赛后按市场盈亏更新余额，以及制作外部公开版下注建议网页时使用。
- `references/account2-script-based-polymarket-execution.md`：主人提供 account2 Python 脚本而非标准 env，并要求直接用脚本或脚本中的常量执行 live 测试/下单时使用；包含 create_or_derive_api_key 警告但订单仍可成功的处理、HK 执行、positions/activity 双重复核和脱敏报告。
- `references/polymarket-pool-participant-allocation-accounting.md`：多人资金池参与下注时使用；覆盖本金修正、利润分配、份额转让、池子占比、以及按每个具体 market/outcome 将 account1/account2 持仓成本分摊到每个人的账务口径。
- `references/pool-ledger-and-participant-allocation.md`：用户要求记录参与人、纠正本金、让出份额、分配已实现利润、按池子占比分摊当前 account 持仓时使用；区分资金池余额、已实现利润分配和未平仓 betting 成本分摊。
- `references/polymarket-participant-pool-accounting.md`：记录多人参与下注池、本金、当前账面金额、某场/本轮利润按指定比例分配、每人余额和池子占比时使用。
- `references/polymarket-multi-account-auth-troubleshooting.md`：account2/multi-account 认证失败或用户怀疑误用 account1 API 时使用；包含脱敏 env 对比、脚本读取路径核验、signature_type/funder auth matrix、截图密钥核对注意事项和 1 USDC live test 安全门。
- `references/live-fixture-market-api-probes.md`：早盘/临场只读核验探针；包含 FIFA calendar、ESPN scoreboard/summary、Polymarket Gamma keyset More Markets、CLOB orderbook 的可复用 API 路径与输出注意。
- `references/moltbook-x-football-prediction-learning-20260613.md`：从 Moltbook/ClawHub/X 搜索中吸收的足球预测增强框架；包含盘口/基本面/阵容/防守伤停/xG/确定性分级/仓位版式要求。Cron 生成“确定性分级卡片”时必须参考。
- `references/worldcup-x-signal-monitor-wiring-20260613.md`：复用旧 X monitor skills/scripts 的世界杯 X 信号监控接线；包含 runner/config/state/error 路径、当前 X API 402 限制和 cron 使用方式。
- `references/worldcup-certainty-card-web-cron-20260613.md`：用户确认的“世界杯确定性分级卡片”网页/长图/cron 交付规范；强调不要混用教育页、不要回到球员海报风格、固定 URL 与验证步骤。
- `references/inplay-position-pnl-audit-and-risk-adjustment-20260613.md`：本场会话沉淀的临场调仓与真实 PnL 审计规则；强调真实盈亏必须读 account positions/activity，部分平仓用 SELL 现金流 + 剩余 currentValue，live advice 先查 orderbook best bid/ask。
- `references/injury-driven-lineup-reassessment-20260613.md`：临场伤停/首发新情报进入已有下注计划时使用；按伤病影响位置重跑 market thesis，把旧方向明确标成升级/保留/降级/取消，并给出已有持仓减仓/对冲比例，例如 Brazil vs Morocco 小2.5 从主线降为保护仓、Qatar vs Switzerland 清洁伤停但不追深盘。
- `references/matchday-injury-adjusted-public-page-and-risk-downgrade-20260613.md`：伤停信息导致 4 场计划重排、用户纠正多腿执行过于积极、白色长图网页替换深色卡片页、以及 account1 子账户转让的综合复用记录。
- `references/matchday-analysis-long-image-and-public-page-20260613.md`：把世界杯比赛日分析做成长图和公开网页的复用流程；包含报道20字总结、PIL长图、public_betting_page更新、UTF-8验证和Playwright截图复核。
- `references/same-ratio-multi-account-execution-20260613.md`：用户说“同样比例 / account2 也按同样比例”时使用；覆盖按最新已执行 notional 比例缩放、Polymarket BUY 精度报错的整数 size 修正、account1/account2 执行路径差异、以及执行后 positions/activity 回读格式。
- `references/polymarket-order-precision-and-verification-notes-20260613.md`：大额 FOK/BUY 下单精度与复核补充；遇到 maker/taker amount decimals 报错时，调整 size 让 `price*size` 为 2 位小数，优先整数 share，并用 raw CLOB `/book` 兜底核验 orderbook。

量化/套利类 Polymarket 旧方案（如 Yes+No 双腿价格和低于 0.98 时两边下单）不写进世界杯预测主逻辑；需要时跳转 `quant-trading-scripts/references/polymarket-strategy-2-1-dual-leg-arbitrage.md`，先按量化脚本高风险流程只读核查，再决定是否 dry-run/live。
