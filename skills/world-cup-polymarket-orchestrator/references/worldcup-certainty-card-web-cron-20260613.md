# 世界杯确定性分级卡片网页与 cron 交付规范（2026-06-13）

## 本轮用户确认的最终版式

用户明确认可的版本不是球员海报型，也不是旧教育页/课程页，而是“简洁 2x2 确定性分级下注指南卡片”。后续世界杯赛前汇报、长图和 cron 生成应保持这一版式，除非用户明确要求换风格。

固定页面：

```text
https://8-219-130-25.sslip.io/worldcup-future3/
/var/www/worldcup-future3/index.html
```

不要修改或复用：

```text
https://beta.openaitalk.com/
```

该地址对应其它内容；本轮用户明确纠正“我们讨论的不是一个事情，我要的是世界杯的网页地址和长图，不是教育的”。

## 卡片必须包含的字段

每场比赛一张卡片：

- 比赛名：如 `USA vs Paraguay`
- 推荐投入：如 `1000 USDC`
- 确定性等级：A / A- / B+ / B / C
- 确定性分数：0-100
- 主方向：如 `USA Yes`
- 副方向：如 `Under 2.5`
- 判断说明：简短中文，不堆长文
- 降级/取消条件：首发、伤停、盘口、流动性、X/新闻信号异常
- 底部统一风控规则：确定性不是必胜；不要重仓串关；盘口/首发大变则降级

用户认可的示例结构：

```text
USA vs Paraguay
推荐投入：1000 USDC
确定性：A / 82
主方向：USA Yes 600
副方向：Under 2.5 400
判断：美国主场与整体冲击力占优，小球保护比分波动。
```

## 交付顺序

1. 更新 `/var/www/worldcup-future3/index.html`。
2. 验证固定 URL 返回 HTTP 200，页面标题应体现“世界杯…下注方案与确定性分级”。
3. 用 Playwright/Chromium 截取 full_page 长图，宽度 1080。
4. 长图保存到：

```text
/root/.hermes/media_cache/worldcup_webpage_YYYYMMDD/
```

5. 返回 Telegram 时直接附 `MEDIA:` 长图，并给固定网页地址。
6. 不要只说“已生成”而不验证；必须有文件尺寸/存在性验证。

## Cron 规则

已建立的赛前任务：

```text
job_id: 0218437cac84
name: worldcup-pre-match-certainty-guide
frequency: every 30m
trigger window: future 45–75 minutes before kickoff
```

运行逻辑：

- 无比赛：静默，不解释。
- 有比赛：生成最新网页 + 长图 + Telegram 简报。
- 每次临场先跑 X 信号监控：`/root/.hermes/scripts/worldcup_x_signal_monitor.sh`。
- X/RSS/API 没有可靠数据时，不阻塞；页面写“X 实时信号暂无可靠数据”。

## 易错点

- 不要把其它网页/课程页/教育页当作世界杯网页来改。
- 不要回到球员海报型输出；用户已确认简洁卡片版更有道理。
- 不要让旧 lhr.life 临时地址成为主交付地址；主地址用固定 sslip 页面。
- 不要把 X 未验证传闻当事实；只作为线索，需交叉验证。
- 不自动下注，不做资金操作；真实下单需要用户再次明确确认。
