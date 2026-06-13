---
name: world-cup-caiqiu-prediction
description: 世界杯彩球/足球赛事预测与手动下注辅助工作流：按赛程、球队状态、伤停、赔率、盘口、Polymarket 市场和资金风控，生成逐场执行卡、条件单、观望单与复盘记录。只做研究参考，不自动下注。
tags: [world-cup, football, soccer, caiqiu, betting, polymarket, prediction, risk-management]
version: 2.0.0 (calibration patch)
author: Hermes Agent
---

# 世界杯彩球预测 Skill

> 【v2 摘要】预测/校准/风控部分套用与 daily-prediction / risk v2 相同的升级：
> 最低 edge 门槛、份数改由 edge 推导（指向风控 skill）、回撤/置信区间停手、
> Brier/CLV 校准账本 + Kill-switch、slug/价格数据校验、先估概率后看盘口。
> **实盘执行/HK 路由部分本版只加"执行前刹车"，不优化其机制**（见安全边界 §2、查询规则 §8）。
> 目标是少亏、不自欺，不是提高胜率。

## 适用场景

当用户说：
- “世界杯彩球”
- “世界杯怎么下 / 足球怎么下 / 今日足球单”
- “给我做世界杯预测 / Polymarket 链接 / 彩球策略”
- “每天早晚更新世界杯下注方向”

使用本 skill 输出**逐场可执行但保守风控**的研究参考。

Session-derived Polymarket 执行与比例下注细节见：`references/2026-polymarket-world-cup-proportional-staking.md`。

## 安全边界

1. 默认只做研究与手动下注参考，不自动下单。
2. 若用户**明确要求实盘下单**（例如指定“下注/买入/每单 X USDC”），允许进入受控执行模式，但必须先完成：HK 出口验证、余额/allowance 检查、有效 `clobTokenId` + orderbook 验证、金额上限检查，并在结果中回报 orderID/tx/status；不得在聊天中回显私钥/API secret/passphrase。
   - **【v2 刹车】进入受控执行模式前，必须先通过校准门与数据完整性门**
     （见"校准账本与 Kill-switch"与查询规则 §10）：
     若校准 Kill-switch 触发（模型未跑赢市场基准且平均 CLV ≤ 0）→ 实盘执行 `BLOCKED`，
     仅允许 dry-run / 研究。
   - **【v2 范围声明】**本版不优化 HK 出口路由本身的实现。注意：本条允许的"受控执行模式"
     与 §1"只做研究参考"、以及 `world-cup-daily-prediction` 4.1 第 9 条"不要规避地区限制"存在张力；
     HK 路由与多账户/资金池属合规问题，请由有资质的专业人士评估，不作为纯技术项处理。
3. 禁止使用“稳胆、必中、稳赚、梭哈、包赢”等确定性语言。
4. 不鼓励赌博；若信息不足，优先输出“观望/不碰”。
5. 不追单、不倍投、不用马丁格尔。
6. 娱乐单不能伪装成主单。

## 必须加载/兼容的相关 skills

建议同时遵循：

- `sports-betting-risk-management`（**【v2】仓位与停手以此为唯一来源**）
- `world-cup-daily-prediction`（**【v2】共享同一校准账本**）

若用于 Polymarket 查询，也可参考：

- `research-source-monitoring`
- `quant-trading-scripts` 中的只读 Polymarket 工具

## 数据核验优先级

每次输出前必须查当前事实，不得凭记忆：

1. 赛程/开球时间：FIFA、赛事官网、ESPN、BBC、Sofascore、FotMob。
2. 已完赛结果：ESPN、BBC、Sofascore、Flashscore。
3. 阵容/伤停：球队官网、赛事官网、Sofascore/FotMob lineup、主流体育媒体。
4. 积分/晋级形势：赛事官网或可靠体育数据源。
5. 赔率/盘口：OddsPortal、主流赔率聚合源、Polymarket Gamma API。
6. 天气/场地：仅在明显影响比赛时使用。

如果无法获取可靠数据，必须明确写：`暂无可靠实时数据`，改用条件阈值，不得编造。

**【v2】数据完整性校验（每场强制）**：
- 赛程交叉核验对阵/日期/分组，对不上 → `FIXTURE_MISMATCH`，跳过该场；
- Polymarket slug 队伍三字码必须与对阵匹配（防 `ger-kor` 类错链）→ 不匹配标 `SLUG_MISMATCH`，撤链；
- 每个价格带拉取时间戳，距今 > 15 分钟 → `STALE_PRICE`，使用前重拉。

## 每日两次更新模式

### 04:00 早盘版

用途：隔夜复盘 + 未来 24-36 小时初版预测。

必须包含：
- 过去 24 小时复盘；
- 今日/明日赛程核验；
- 初版主方向、副方向；
- 赔率阈值；
- 需要 16:00 临场复核的变量。

标题：

```text
世界杯彩球早盘更新（北京时间 YYYY-MM-DD 04:00）
```

### 16:00 临场版

用途：当天执行前二次确认。

必须包含：
- 当天剩余比赛核验；
- 首发/伤停/盘口/Polymarket 价格变化；
- 对早盘判断做“上调/维持/降级/取消”；
- 最终主单、条件单、观望单。

标题：

```text
世界杯彩球临场执行更新（北京时间 YYYY-MM-DD 16:00）
```

## 概率估计与价值判断（v2 调整）

- **【v2】先估概率，后看盘口**：进入比较前，先独立估出主胜/平/客胜与 O/U、BTTS 的模型概率，
  写明依据（Elo/排名/xG 差 + 主中立场/伤停/赛制调整，每项注明加减幅度）。
  若模型概率与市价几乎相同，标"疑似锚定，复核 q 独立性"。
- **【v2】价值差用买入价、并设门槛**：`价值差 = 模型概率 − 买入价隐含概率`（用卖一/买入价，含价差）；
  **仅 `价值差 ≥ 5 个百分点` 才可标为可执行**，否则写"无 edge / 不下"。1–2pp 视为噪声。

## 官方首发确认窗口（v2.1 临场规则）

- 球队提交首发表给当值官员通常在赛前 75-90 分钟，但公众可见的官方确认 XI 一般在赛前约 60 分钟。
- 临场 cron/check 推荐开球前 70 分钟先启动，主盯 RotoWire 世界杯首发聚合页 `https://www.rotowire.com/soccer/lineups.php?league=WOC`（一个 URL 覆盖全部比赛；未出时显示 `lineup has not been posted yet`；出后填 confirmed XI 并标 OUT/QUES），同时盯 FIFA/FIFA.com、两队官方社媒/官网、Sofascore/FotMob/Flashscore 推送。
- RotoWire World Cup lineups、lineups.com、WhoScored、ESPN 可用于预测/确认对照；但未官方确认前一律标 `PROVISIONAL`。
- 只有 FIFA/球队官方/可靠聚合源明确 confirmed XI 后才标 `CONFIRMED`，并立即重估 q、重拉 ask p、重跑 edge gate，输出上调/维持/降级/取消。
- 若 T-55 仍未确认，标 `LINEUP_NOT_CONFIRMED`，首发乘数最多 0.5，只能维持保守或降级，不得上调仓位。
- 每场卡片必须列：`Lineup status / source / timestamp / changed players / new q / ask p / edge / action`。

## 仓位纪律

以 100 单位为总资金，用户说“1份/一份”时默认把 1 份作为基准仓位（例如用户当前指定 1 份=10 USDC），但**不得所有比赛机械等额下注**。

**【v2】仓位与停手以 `sports-betting-risk-management` v2 为唯一来源，本 skill 不再复述数字**：
- 份数由 **edge 推导**（1/4 凯利：`base = 0.25 × (q−p)/(1−p)`），信心/首发只作收缩，不上抬；
- 下面的档位区间**仅作 sanity 上限**，不是 sizing 规则：
  C=0；B- ≤0.5份；B ≤0.8份；B+ ≤1.0份；A- ≤1.2份；A ≤1.5份；
- 单场总风险上限 2 份（胜负+大小球+其他合计）；
- **【v2】跨场同方向上限**：同方向（CHALK/UNDER/…）跨场合计 ≤ 总资金 5%；
- 单日建议总投入 3-5 份以内；
- **【v2】停手用回撤/置信区间，弃"连黑3场"硬停**：
  单日回撤 3% 停新增；峰值回撤 8% 全仓减半；滚动 50 注落在模型 95% 区间外则暂停排查。
  （"连续错 3 场"仅作提示复核，不作硬停——那是方差不是信号。）
- 正确比分、半全场、串关：0.1-0.2 份娱乐，期望为负、单独标注、不进主仓、不计 edge。

### 比例下注输出规则

每场必须输出：

```text
信心等级：A/B/C
价值等级：高/中/低
模型倾向概率：...%        （【v2】先于看市价估出）
市场隐含概率：...%        （【v2】用买入价/卖一，含价差）
价值差：... 个百分点       （【v2】<5pp 即 pass）
建议份数：按 edge 推导（见风控 skill；本字段不写死整份）
仓位理由：为什么比其他场更大/更小
反证：最强反方观点 / 推翻条件 / 降级触发器
```

分配原则：
- 信心强但赔率太低：降低份数，不追高价；
- 信心一般但赔率有价值：中小份，不重仓；
- 胜负方向和大小球若逻辑高度相关，合计不能超过单场上限；
- 大小球通常作为副线，默认不得超过该场主方向，除非大小球信号更强；
- 已有持仓后补单时，只补到目标份数，不机械再买一整份。

## 比赛分级

### A 级：可执行

条件：
- 赛程、阵容、动机、盘口较清晰；
- 赔率有价值，没有被压穿；**【v2】价值差 ≥ 5pp**；
- 风险因素可控。

### B 级：条件执行

条件：
- 有方向，但赔率/首发/伤停仍需确认；
- 只在达到阈值时执行。

### C 级：观望

条件：
- 动机复杂、轮换风险大；
- 盘口过热；
- 信息缺失；
- 市场价格没有价值（**【v2】价值差 < 5pp**）。

## 每场输出模板

```text
比赛：A vs B
时间：北京时间 YYYY-MM-DD HH:MM
阶段：小组赛/淘汰赛/其他
风险等级：A/B/C
【v2】数据校验：FIXTURE ✓ / SLUG ✓ / PRICE(ts=hh:mm) ✓

评分摘要：基本面/阵容/动机/市场价格/信息确定性 = ... /25
模型倾向概率：...        （先于看市价）
市场隐含概率：...        （买入价/卖一）
价值差：...              （≥5pp 才可执行）
结论：...
主方向：...
副方向：...
赔率阈值：...
仓位：按 edge 推导（见风控 skill）
Polymarket：市场标题 / Yes 或 No / 当前价格或概率 / 链接；找不到则写未找到直接市场
不碰：...
反证：...
临场复核：首发、伤停、盘口、天气、动机
如果临场变化：...
```

## 校准账本与 Kill-switch（v2 新增，与 daily-prediction 共享）

每条 `CONFIRMED` 预测写入共享账本：
`日期 / 比赛 / market / 模型概率 q / 买入价 p / 结果 / 收盘价 / CLV`。

滚动 50 注计算并写入复盘：
- **Brier(模型)** vs **Brier(跟随市价基准)**；
- **平均 CLV**（>0 才是 edge 最可靠的证据）。

**Kill-switch（硬规则）**：模型 Brier ≥ 基准 且 平均 CLV ≤ 0 → 停止据此真金下注（含实盘执行 BLOCKED），
回研究模式直到重新通过；仅边际优于基准 → 额外 ×0.5。
风控 skill 与安全边界 §2 的执行刹车都读取此结果。

## Polymarket 查询与执行规则

1. 用球队英文名、赛事名、日期搜索 Polymarket/Gamma markets。
2. 优先给直接对应比赛的 event/market 页面。
3. 优先解析 Gamma API 的 `outcomes`、`outcomePrices`、`clobTokenIds`，注意它们可能是 JSON 字符串；`clobTokenIds` 与 `outcomes` 顺序一一对应。
4. 对世界杯 games 页面，普通搜索可能漏掉 child/More Markets。必要时用 Gamma keyset：`/events/keyset?tag_id=102232&series_id=11433&closed=false&order=startTime&ascending=true&limit=100&include_best_lines=true`。
5. 大小球：`totals` 市场中 `outcomes[0]/clobTokenIds[0]` 是 Yes/Over，`outcomes[1]/clobTokenIds[1]` 是 No/Under；买 Under 2.5 应买 `line=2.5` 的 No token。
6. 每个链接必须写清：市场标题、建议方向 Yes/No/观望、当前价格/概率、执行阈值。
7. 找不到直接市场时写：`Polymarket：未找到该场直接市场`，不要伪造链接。
8. 实盘下注只在用户明确授权时执行，并且必须从用户指定香港服务器执行。当前已验证工作目录：`<YOUR_HK_SERVER_IP>:/opt/polymarket-worldcup`；执行前必须 `orderbook` 验证 token 有效，`balance` 验证余额/allowance，且 live 下单命令必须具备 HK 出口 guard。
   - **【v2】本版不优化此条的 HK 执行机制**；仅追加要求：执行前必须先过 §10 校验门与校准门，
     任一不过即 abort。HK 路由/多账户/资金池的合规性请由专业人士评估。
9. 详细实盘执行参考：`references/polymarket-live-execution-hk-2026-06-11.md`。
10. **【v2】执行前校验门（任一失败即 abort 该腿）**：
    - SLUG 队伍码匹配；`clobTokenId` 取自当前 Gamma 数据且 orderbook 有效；
    - 价格时间戳 ≤ 15min；SIZE 精度合规、份额不跨市场相加；
    - 校准 Kill-switch 未触发（否则只允许 dry-run）。

## 输出结构

```text
世界杯彩球早盘/临场更新（北京时间 YYYY-MM-DD HH:MM）

结论：今天保守/正常/进攻；最多投入 X 单位；亏损到 Y 单位停手。

一、过去 24 小时复盘
- 命中/偏差：...
- 模型修正：...
- 【v2】滚动 Brier(模型 vs 基准) 与 平均 CLV：...

二、未来 24-36 小时比赛总览
- A vs B：主方向 / 风险等级 / 仓位 / 是否需临场复核

三、逐场执行卡
1. ...

四、最终单
主单：...
条件单：...
观望/放弃：...
娱乐单：...

五、风控
不追单、不倍投、不重仓比分、不因上一场输赢改变仓位。
```

## 质量检查

输出前确认：

- 赛程和时间已核验并换算北京时间；
- 每场都有方向、仓位、赔率阈值、不碰项；
- 每场都区分了“预测概率”和“下注价值”，并给出价值差或放弃原因；
- **【v2】价值差用买入价计算，且仅在 ≥5pp 时标可执行；模型概率先于市价估出；**
- **【v2】每场有数据校验状态（FIXTURE/SLUG/PRICE）；slug 未过校验的链接已撤下；**
- A/B 级方向都有反证、推翻条件和临场降级触发器；
- Polymarket 找不到时明确说明，不伪造；
- 有总投入上限和停手机制（**【v2】引用风控 skill；停手用回撤/区间，非连败计数**）；
- **【v2】复盘含滚动 Brier 与平均 CLV；Kill-switch 状态已检查；**
- 没有“必胜/稳赚/稳胆”等措辞。

## Cron 推荐配置

每天北京时间 04:00 与 16:00：

```cron
0 4,16 * * *
```

推荐加载工具集：

- `web`
- `browser`
- `terminal`
- `skills`
- `tts`

推荐 prompt：

```text
请加载并遵循 world-cup-caiqiu-prediction、world-cup-daily-prediction、sports-betting-risk-management。
核验未来 24-36 小时世界杯/足球大赛赛程、结果、积分、伤停、赔率与 Polymarket 市场。
按当前北京时间判断早盘版或临场版，输出逐场执行卡、主单/条件单/观望单和风控。
只做研究参考，不自动下注，不使用稳赚/必中/稳胆语言。
```
