# Orchestrator workflow

## 目标

把四个 skill 串成一个稳定流水线：

1. `research-source-monitoring`：查赛程、信息源、Polymarket 市场。
2. `world-cup-daily-prediction`：做 04:00 早盘与 16:00 临场判断。
3. `sports-betting-risk-management`：给信心等级、赔率价值、仓位比例、止损。
4. `world-cup-caiqiu-prediction`：输出逐场执行卡并在用户明确授权时进入 Polymarket 执行。

## 运行顺序

```text
查询赛程和事件
→ 查询市场和价格
→ 判断主方向/副方向
→ 给 A/B/C 信心与价值等级
→ 计算目标份数和本次补仓
→ 验证 clobTokenId / orderbook / balance / HK
→ live 下单或给手动执行卡
→ 成交回报和赛后复盘
```

## 输出要求

每场都必须给：

- 比赛与时间；
- Polymarket 事件/市场链接；
- 主方向与副方向；
- 信心等级；
- 价值等级；
- 目标份数；
- 现有持仓；
- 本次补单；
- 执行阈值；
- 风险与取消条件。

## 何时不执行

- 赛程/市场无法可靠核验；
- `clobTokenId` 无 orderbook；
- 非 HK 出口；
- 余额/allowance 不足；
- 价格已超过阈值；
- 当天已触发止损或连错停止；
- 主人没有明确要求实盘。
