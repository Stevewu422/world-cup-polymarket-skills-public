# World Cup X signal monitor wiring（2026-06-13）

## 已启用的本地监控脚本

- Combined runner: `/root/.hermes/scripts/worldcup_x_signal_monitor.sh`
- RSS runner: `/root/.hermes/scripts/worldcup_x_signal_monitor_rss.sh`
- RSS config/state/output/error:
  - `/root/.hermes/private_tasks/coco/worldcup_x_monitor/config_rss.json`
  - `/root/.hermes/private_tasks/coco/worldcup_x_monitor/state_rss.json`
  - `/root/.hermes/private_tasks/coco/worldcup_x_monitor/new_items_rss.txt`
  - `/root/.hermes/private_tasks/coco/worldcup_x_monitor/last_error_rss.log`
- API v2 config/state/output/error:
  - `/root/.hermes/private_tasks/coco/worldcup_x_monitor/config_api_v2.json`
  - `/root/.hermes/private_tasks/coco/worldcup_x_monitor/state_api_v2.json`
  - `/root/.hermes/private_tasks/coco/worldcup_x_monitor/new_items.txt`
  - `/root/.hermes/private_tasks/coco/worldcup_x_monitor/last_error_api_v2.log`
- Core scripts reused from prior X monitoring skill:
  - `/root/.hermes/skills/openclaw-shared/news-monitoring-scripts/scripts/x_monitor/x_monitor_rss.py`
  - `/root/.hermes/skills/openclaw-shared/news-monitoring-scripts/scripts/x_monitor/x_monitor_api_v2.py`

## 当前监控目标

Config includes:

- Official/near-official accounts:
  - `FIFAWorldCup`
  - `FIFAcom`
  - `Polymarket`
  - `OptaJoe`
- Search queries:
  - `(World Cup OR FIFA World Cup) + Polymarket/odds/betting/prediction/lineup/injury/xG`
  - current match-pair queries such as USA vs Paraguay / Canada vs Bosnia / Brazil vs Morocco / Haiti vs Scotland

## 已验证问题

- `/root/.openclaw/secrets/x.env` 存在 X credentials，但直接 `source` 会因 `X_SCOPE` 含空格报错；runner 已改为只安全读取 `X_BEARER_TOKEN`。
- 当前 X API recent search/user endpoints 返回 HTTP 402（plan/access 限制），所以脚本可能静默无新项并在 `last_error_api_v2.log` 写入 402。
- 之前的 RSS/Nitter 路线也不稳定：nitter.net 断连，多数实例 403/HTML challenge/证书错误；xcancel RSS 需要 whitelist。

## Cron 使用方式

`worldcup-pre-match-certainty-guide` cron 每次临场生成前应运行：

```bash
/root/.hermes/scripts/worldcup_x_signal_monitor.sh
```

- 若 stdout 有新 X items：作为临场舆情/伤停/阵容/盘口信号输入，并在页面中引用链接。
- 若 stdout 为空但 error log 有 402/403/timeout：不要阻塞；页面中写“X 实时信号暂无可靠数据”。
- 不要把未验证 X 传闻当成确定事实；必须与 FIFA/ESPN/FotMob/Sofascore/Polymarket market data 交叉验证。

## Hourly news-report monitor

A separate non-trading news report can run independently from the pre-match betting guide:

- Script: `/root/.hermes/scripts/worldcup_news_hourly_report.py`
- State: `/root/.hermes/private_tasks/coco/worldcup_news_monitor/state.json`
- Cron pattern: `cronjob(no_agent=true, script="worldcup_news_hourly_report.py", schedule="every 1h", deliver="origin")`
- Current job created in this session: `worldcup-news-hourly-report` / `ca8ffdb381b8`.

Behavior:

- Pulls Google News RSS queries for World Cup topics.
- Calls the combined X signal monitor as an auxiliary source.
- Prints a Telegram-ready Chinese report on stdout.
- Keeps betting/execution separate: this hourly monitor is informational only and must not place orders.
- If X/RSS is unavailable, it still reports Google News items and clearly states “X/RSS信号暂无可靠新内容”.

## 后续改进

- 若升级 X API 访问计划或拿到可用 RSS/Nitter/RSSHub 源，直接更新 `config_api_v2.json` 或增加 RSS runner。
- 若主人指定具体 X 账号/分析师账号，加入 `api_v2_user` sources 或关键词 query。
