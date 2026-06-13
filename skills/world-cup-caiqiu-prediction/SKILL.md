---
name: world-cup-caiqiu-prediction
description: 世界杯彩球/足球赛事预测与手动下注辅助工作流：按赛程、球队状态、伤停、赔率、盘口、Polymarket 市场和资金风控，生成逐场执行卡、条件单、观望单与复盘记录。只做研究参考，不自动下注。
tags: [world-cup, football, soccer, caiqiu, betting, polymarket, prediction, risk-management]
version: 1.0.0
author: Hermes Agent
---

# 世界杯彩球预测 Skill

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
3. 禁止使用“稳胆、必中、稳赚、梭哈、包赢”等确定性语言。
4. 不鼓励赌博；若信息不足，优先输出“观望/不碰”。
5. 不追单、不倍投、不用马丁格尔。
6. 娱乐单不能伪装成主单。

## 必须加载/兼容的相关 skills

建议同时遵循：

- `sports-betting-risk-management`
- `world-cup-daily-prediction`

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

## 仓位纪律

以 100 单位为总资金，用户说“1份/一份”时默认把 1 份作为基准仓位（例如用户当前指定 1 份=10 USDC），但**不得所有比赛机械等额下注**，必须按信心和赔率价值给比例。

- C 级：0 单位，只看不买；
- B- 级：0.3-0.5 份，信息不足/波动较大；
- B 级：0.5-0.8 份，方向有小优势但风险明显；
- B+ 级：0.8-1.0 份，方向较清晰但价格一般；
- A- 级：1.0-1.2 份，方向清晰且价格可接受；
- A 级：1.2-1.5 份，强信号+价格有价值；
- 单场总风险上限：2 份（胜负+大小球+其他市场合计）；
- 单日建议总投入：3-5 份以内；
- 当天亏损 5 份停止；
- 连续错 3 场停止；
- 正确比分、半全场、串关：0.1-0.2 份娱乐，不进入主仓位。

### 比例下注输出规则

每场必须输出：

```text
信心等级：A/B/C
价值等级：高/中/低
建议份数：X 份（若 1份=10 USDC，则约 X*10 USDC）
仓位理由：为什么比其他场更大/更小
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
- 赔率有价值，没有被压穿；
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
- 市场价格没有价值。

## 每场输出模板

```text
比赛：A vs B
时间：北京时间 YYYY-MM-DD HH:MM
阶段：小组赛/淘汰赛/其他
风险等级：A/B/C

结论：...
主方向：...
副方向：...
赔率阈值：...
仓位：... 单位
Polymarket：市场标题 / Yes 或 No / 当前价格或概率 / 链接；找不到则写未找到直接市场
不碰：...
临场复核：首发、伤停、盘口、天气、动机
如果临场变化：...
```

## Polymarket 查询与执行规则

1. 用球队英文名、赛事名、日期搜索 Polymarket/Gamma markets。
2. 优先给直接对应比赛的 event/market 页面。
3. 优先解析 Gamma API 的 `outcomes`、`outcomePrices`、`clobTokenIds`，注意它们可能是 JSON 字符串；`clobTokenIds` 与 `outcomes` 顺序一一对应。
4. 对世界杯 games 页面，普通搜索可能漏掉 child/More Markets。必要时用 Gamma keyset：`/events/keyset?tag_id=102232&series_id=11433&closed=false&order=startTime&ascending=true&limit=100&include_best_lines=true`。
5. 大小球：`totals` 市场中 `outcomes[0]/clobTokenIds[0]` 是 Yes/Over，`outcomes[1]/clobTokenIds[1]` 是 No/Under；买 Under 2.5 应买 `line=2.5` 的 No token。
6. 每个链接必须写清：市场标题、建议方向 Yes/No/观望、当前价格/概率、执行阈值。
7. 找不到直接市场时写：`Polymarket：未找到该场直接市场`，不要伪造链接。
8. 实盘下注只在用户明确授权时执行，并且必须从用户指定香港服务器执行。当前已验证工作目录：`<YOUR_HK_SERVER_IP>:/opt/polymarket-worldcup`；执行前必须 `orderbook` 验证 token 有效，`balance` 验证余额/allowance，且 live 下单命令必须具备 HK 出口 guard。
9. 详细实盘执行参考：`references/polymarket-live-execution-hk-2026-06-11.md`。

## 输出结构

```text
世界杯彩球早盘/临场更新（北京时间 YYYY-MM-DD HH:MM）

结论：今天保守/正常/进攻；最多投入 X 单位；亏损到 Y 单位停手。

一、过去 24 小时复盘
- 命中/偏差：...
- 模型修正：...

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
- Polymarket 找不到时明确说明，不伪造；
- 有总投入上限和停手机制；
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
