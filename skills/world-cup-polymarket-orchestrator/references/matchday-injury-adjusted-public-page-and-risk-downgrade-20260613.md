# 2026-06-13 世界杯伤停修正版、网页长图与风险降级经验

## 触发场景

用户连续补充 4 场未来 24 小时比赛的伤停/首发信息，并要求重新预判、重画长图、更新公开网页，同时涉及 Polymarket account1/account2 持仓修正和 account1 资金池子账户转让。

## 可复用经验

### 1. 新伤停信息到来后先重新定级，再谈执行

不要沿用旧组合计划。每条腿按 `保留 / 降级 / 退出 / 新增观察` 重新标注：

- 瑞士 vs 卡塔尔：伤停干净，瑞士胜保留，瑞士 -1.5 只小仓，不追 -2.5。
- 巴西 vs 摩洛哥：巴西缺内马尔和右后卫、摩洛哥缺中卫但进攻完整；Under 2.5 从主线降为保护仓，关注 BTTS Yes / Over 2.5 对冲条件。
- 海地 vs 苏格兰：McTominay 可用利好苏格兰胜，但 Bellegarde 信息打架；苏格兰胜保留，Scotland -1.5 与 Over 2.5 降为小仓/观察。
- 澳大利亚 vs 土耳其：澳大利亚满血低位，土耳其 Çalhanoğlu/Yıldız/Ferdi 临场存疑；Under 2.5 优先，Turkey win 降级，Turkey -1.5 不碰，等 Çalhanoğlu 首发。

### 2. 不要把模糊确认理解为执行所有相关腿

本次用户质疑“你为撒会执行 -1.5 和 O2.5 两个呢？”说明：当计划里包含 win + spread + total 多条相关市场时，必须要求明确执行口令，尤其当新伤停信息已经削弱某些腿时。推荐口令形如：

```text
执行A：两个账户都减半 Brazil Under 2.5
执行苏格兰降风险：-1.5 卖一半，大2.5 卖三分之一
只执行苏格兰胜，不执行 -1.5/O2.5
```

### 3. 白色长图网页交付偏好

用户不喜欢深色卡片网页时，会说“不要这个版本，需要白色图片那种的版本”。处理方式：

1. 将已生成的白底长图 PNG 上传到公网页面 assets，例如 `public_betting_page/assets/worldcup_next4_injury_adjusted_white.png`。
2. 把首页改成简单白色 wrapper + `<img>`，不要重写成深色 HTML 卡片。
3. 给图片加 cache-busting query，比如 `?v=YYYYMMDDHHMM`。
4. 同时验证 HTML URL 和 PNG URL HTTP 200，并用截图确认手机可浏览。
5. 用非数字 IP 的可浏览地址，例如 `http://47-239-251-66.sslip.io/`，不要只给裸 IP。

### 4. account1 子账户转让口径

当用户说“Steve 转让 1000USD 给新子账户 zeng hui”：

- 这是 account1 资金池内部转让，池子总额不变。
- Steve 余额减少 1000，zeng hui 增加 1000。
- 重新计算后续下注占比。
- 历史已下注市场仍按下注时快照占比分摊，不能用新占比重算旧仓位。
- 修改 `latest_account1_pool.md` 前先备份，并在账本顶部记录转让说明。

## 验证清单

- 盘口：CLOB best bid/ask 重新拉取。
- 持仓：`data-api.polymarket.com/positions` 回读 size、avgPrice、initialValue、currentValue、cashPnl。
- 活动：`data-api.polymarket.com/activity` 确认手动 SELL 或 BUY。
- 网页：公网 HTTP 200；中文搜索时设置 UTF-8；白色 PNG URL 200；截图清晰。
- 资金池：余额合计等于池子总额，占比合计约 100%。
