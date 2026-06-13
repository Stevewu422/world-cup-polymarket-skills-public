# Polymarket execution checklist

## 必须执行服务器

- Host: `<YOUR_HK_SERVER_IP>`
- Workdir: `/opt/polymarket-worldcup`
- 所有真钱 `post_order` 必须从香港服务器执行。

## 执行前检查

1. `country == HK` / `city == Hong Kong`。
2. market active=true 且 closed=false。
3. 从 Gamma 当前数据获取 `clobTokenId`，不要使用旧脚本/截图 token。
4. `outcomes[i]` 与 `clobTokenIds[i]` 顺序一致。
5. 下单 token 的 orderbook 查询成功。
6. balance 和 allowance 足够。
7. 本次 notional 未超过仓位计划和 `POLYMARKET_MAX_ORDER_USD`。
   - 默认 `.env` 可保持小额保护值；用户明确授权的大额加仓可用单次命令环境变量覆盖，例如 `POLYMARKET_MAX_ORDER_USD=700 python3 pm_worldcup.py place ...`，不要为了单次交易长期改 `.env`。
8. BUY size 精度符合 CLOB 要求；大额 BUY 若遇到 `invalid amounts ... maker amount ... taker amount ... accuracy`，优先把 `--size` 四舍五入为整数份额后重试 FOK。
9. 私钥、API secret、passphrase 不得输出到聊天或日志摘要。

## 大小球

- totals `line=2.5`：通常 `Yes=Over`，`No=Under`。
- 买 Under 2.5 前必须确认当前 `outcomes` 实际含义，不靠默认猜。

## 成交回报

只报告非敏感字段：

```text
market
outcome
limit price
size
notional
status
orderID
transactionHash
before_balance
after_balance
```

## 失败处理

- `invalid token id`：重新从 Gamma 获取市场；
- `No orderbook exists`：不下单；
- `invalid order version`：用户上传脚本 SDK 旧，用已验证 v2 helper；
- `insufficient balance/allowance`：停止并报告；
- 非 HK：停止并报告，不绕过。
