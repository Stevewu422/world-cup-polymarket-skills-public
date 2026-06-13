# 世界杯赛前预测增强：Moltbook/ClawHub/X 学习摘录（2026-06-13）

## 来源与验证状态

- Moltbook API 搜索：`football prediction betting`、`soccer match prediction`、`sports betting bankroll`、`prediction markets sports`、`Polymarket football`、`world cup betting prediction`。
- ClawHub inspected skills：`ballball`、`soccer-predict`、`hergunmac`、`polymarket-football-odds`、`polymarket-sports-live-trader`、`world-cup-2026-odds`、`polymarket-odds`、`sports-skills-polymarket`。
- X 搜索尝试：`x.com/search` 超时；Nitter 断连；xcancel RSS 要 whitelist；RSSHub twitter keyword 返回 404。本轮未得到可直接引用的可靠 X 帖子内容，不能把 X 作为已核验信号，只保留“后续接入 X 关键词监控”的需求。
- 未直接安装第三方 skill；只吸收公开 metadata / SKILL.md / references 的方法，避免执行未审计代码。

## 可吸收的预测框架

### 1. 赛前数据采集维度（来自 ballball / soccer-predict）

每场临场卡片在 Polymarket 价格之外，新增检查：

- 近期状态：近 5-10 场 W/D/L、进球、失球。
- 主客场/中立场表现：主场胜率、客场胜率、中立赛经验。
- H2H：近 3-5 年交锋与进球趋势；世界杯国家队样本少时标注“参考弱”。
- 战意：小组出线、净胜球、轮换、是否必须抢分。
- 阵容：首发 XI、核心伤停、停赛、替补深度；首发通常开赛前 30-60 分钟确认，未确认时必须写“首发/伤停待确认”。
- 盘口：1X2、让球、大小球；记录开盘与即时盘，临场最终判断使用即时盘。
- 增强数据：xG/xGA、半场进球模式、角球、净胜球分布、天气/场地/休息天数。

### 2. 盘口与概率计算

- 即时盘口优先于早盘；早盘只用于识别升盘/降盘/水位变化。
- 计算去水隐含概率：
  - 双边市场：`P(a)=1/odds_a`，`P(b)=1/odds_b`，`trueP(a)=P(a)/(P(a)+P(b))`。
  - Polymarket：价格本身接近市场隐含概率，但仍需看 bid/ask spread 和 liquidity。
- 盘口走势解释：
  - 升盘：强队被进一步支持，但需防过热。
  - 降盘：强队信心下降或诱盘风险。
  - 只调水不调线：市场分歧或风控调价。
- 欧赔/亚盘/Polymarket 不一致时，标记“盘口分歧”，确定性下调。

### 3. 模型权重建议

胜负/让球方向：

```text
盘口/市场隐含概率 0.35
基本面 0.20
球队状态与实力 0.20
阵容/伤停 0.15-0.20
战意 0.10
```

大小球方向：

```text
xG/xGA 0.12-0.15
盘口/市场隐含概率 0.15
半场进球/角球/攻势数据 0.15-0.16
主客/中立场与近期进球 0.15-0.20
环境/休息天数/赛事阶段 0.10
防守伤停 0.15（GK/CB/DM 缺阵时重点上调大球）
其他 0.05
```

防守伤停对大小球的修正：

- GK 缺阵：倾向大球，约 +0.75~1.0 球影响。
- CB 缺阵：倾向大球，约 +0.5 球影响。
- DM 缺阵：倾向大球，约 +0.25 球影响。
- FB 缺阵：轻微倾向大球，约 +0.1 球影响。
- 多个防守位置缺阵可叠加；不要误判成“防守差所以双方保守小球”。

### 4. 确定性分级与仓位

保留主人认可的网页卡片格式，但每场确定性必须由以下因素共同决定：

- 数据完整度：赛程/首发/伤停/盘口/orderbook 是否齐全。
- 方向一致性：FIFA/ESPN/FotMob/Sofascore、赔率、Polymarket、模型是否同向。
- 价格价值：模型概率 - 市场价格 是否有正差。
- 流动性：bid/ask spread、orderbook 深度，盘口太薄则降级。
- 波动风险：杯赛轮换、红牌风险、天气、战意复杂、球队风格冲突。

建议等级：

- A / 80+：信息齐全、方向一致、价格仍有价值，可做主仓。
- A- / 70-79：方向清楚但有一项风险，半仓。
- B+ / 63-69：有优势但价格一般或信息不完整，小仓。
- B / 55-62：只小仓试错或等待临场确认。
- C / <55：不下注，只观察。

### 5. 从 ClawHub skills 吸收的工具方向

- `polymarket-football-odds`：可把 Polymarket URL 映射到足球 fixture，并用 API-Football/API-SPORTS 给出 1X2 模型概率；若以后配置 `API_FOOTBALL_KEY`，可作为额外校验源。
- `polymarket-odds` / `sports-skills-polymarket`：市场搜索、market details、token_id、orderbook 是必要只读核验；本地现有 Gamma/CLOB 探针仍优先。
- `world-cup-2026-odds`：若配置 `ODDS_API_KEY`，可比较传统 sportsbook 与 Polymarket 的价格差。
- `polymarket-sports-live-trader`：有用思想是 fan loyalty discount 和 peak-event emotion discount。世界杯决赛、热门队（巴西/英格兰/法国/阿根廷等）被情绪追捧时，热门方向确定性要打折，除非价格仍有价值。
- `hergunmac`：可作为辅助预测源，提供 confidence score、1X2、大小球、BTTS、双重机会等；使用时必须说明为第三方模型，不可替代实时数据。

## Cron 应用要求

`worldcup-pre-match-certainty-guide` cron 后续运行时：

1. 继续使用主人认可的“确定性分级卡片”版式。
2. 赛前 1 小时若首发/伤停不可得，页面必须写“待确认”，并下调确定性。
3. 每个建议必须有 `确定性等级 + 分数 + 推荐投入 + 主方向 + 副方向 + 判断说明 + 降级/取消条件`。
4. 若 Polymarket 价格与模型方向不一致，不强行给仓位；可写“观察/取消”。
5. 不自动下单。真实买入仍需主人明确确认。

## 后续可做

- 若主人允许，可单独安装/隔离测试 `polymarket-football-odds`，前提是提供或配置 `API_FOOTBALL_KEY`。
- 建议添加 X 关键词监控源：`Polymarket World Cup`、`World Cup odds`、`soccer betting model`、`xG injuries odds`、球队名 + `lineup/injury/odds`；当前公共 X 抓取不可用，需要可用 Nitter/RSSHub/账号会话后再纳入 cron。
