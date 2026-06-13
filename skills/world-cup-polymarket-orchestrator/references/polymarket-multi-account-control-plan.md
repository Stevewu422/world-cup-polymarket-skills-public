# Polymarket multi-account control plan

## 触发场景

当主人要求“同时控制多个 Polymarket 账户 / 多账户下注 / 多钱包组合风控 / 多账号一键执行”时使用。

## 安全与合规边界

只适用于主人拥有或明确授权管理的账户。不得用于：

- 绕过平台限额、KYC/地域限制或风控；
- 自买自卖、对敲、制造成交量；
- 多账户相互成交或操纵盘口；
- 在非 HK 执行真钱订单。

如果发现同一策略可能导致两个自有账户互相挂相反方向成交，必须阻断并提示。

### 已泄露 private key 的硬停止规则

如果用户把 `POLYMARKET_PRIVATE_KEY`、完整 API secret/passphrase 或可直接签名的脚本内容发到 Telegram/聊天里，该密钥必须视为已泄露：

1. 不把该 private key 写入正式 `.env.accounts/<label>.env`。
2. 不用该 key 执行 `create_order` / `post_order` / 任何真钱签名。
3. 即使用户说“换不了/有难度/坚持执行”，仍只能做只读、dry-run、orderbook 检查、positions 查询，不能 live。
4. 可保存上传脚本为 `account*_source_*.py` 作为来源备查，但保持账户 `enabled=false`。
5. 若用户要继续实盘，要求新钱包/新私钥/新 API credentials，或改用已有安全账户执行。

这是资金安全硬规则，不是风控建议。

## 目标架构

在香港服务器 `/opt/polymarket-worldcup` 下增加多账户层：

```text
configs/accounts.yaml          # 账户清单，只存 env 路径/标签/限额，不存明文密钥
.env.accounts/<label>.env      # 每个账户独立密钥，600 权限
scripts/pm_multi_account.py    # 多账户只读/下单/汇总入口
logs/multi_account/*.json      # 脱敏执行日志
```

`accounts.yaml` 示例：

```yaml
accounts:
  - label: main
    env_file: .env
    role: primary
    enabled: true
    max_single_order_usdc: 300
    max_match_exposure_pct: 0.10
    max_total_exposure_pct: 0.20
  - label: sub1
    env_file: .env.accounts/sub1.env
    role: secondary
    enabled: true
    max_single_order_usdc: 150
    max_match_exposure_pct: 0.05
    max_total_exposure_pct: 0.10
```

## 执行模式

### 1. aggregate / report

只读统计所有账户：

```text
账户余额
账户持仓
按比赛合并成本/估值/浮盈亏
全账户总敞口
同一市场重复/冲突方向检查
```

### 2. dry-run

按总资本和每账户上限分配订单，但不提交：

```text
目标：Brazil 胜 250 USDC
分配：main 150, sub1 100
检查：HK / balance / allowance / orderbook / per-account cap / global cap
```

### 3. live

只有主人明确确认后执行。按账户逐个 post_order，不并发猛打盘口。

默认执行顺序：

1. 主账户下主方向；
2. 子账户补剩余；
3. 每单后等待 1-3 秒；
4. 订单失败不自动换账户追价，先报告；
5. 所有账户执行后统一 positions + activity 复核。

## 订单分配规则

默认按账户可用余额和上限比例分配：

```text
account_weight = min(available_cash, account_cap_remaining)
order_share = target_notional * account_weight / sum(account_weight)
```

必须同时满足：

- 单账户单场不超过该账户 `max_match_exposure_pct`；
- 全账户单场不超过总资本风控阈值；
- 单笔不超过 `max_single_order_usdc`；
- 不在两个账户买同一比赛互相冲突的方向，除非这是明确的对冲/锁盈，并经过二次确认；
- 让球/大小球/胜负高度相关时按同一“场级风险”汇总。

## 多账户报告模板

```text
多账户执行结果

总计划：500 USDC
实际成交：499.98 USDC
账户数：2

按账户：
- main：成交 300 USDC，当前估值 298.5，浮亏 -1.5
- sub1：成交 199.98 USDC，当前估值 199.1，浮亏 -0.88

按比赛：
- Brazil vs Morocco：成本 250，估值 247.8，浮亏 -2.2
- Haiti vs Scotland：成本 249.98，估值 247.6，浮亏 -2.4

风控：
- 全账户总敞口：X / 总资本 Y = Z%
- 最大单账户敞口：...
- 最大单场敞口：...
```

## 实现检查清单

1. 每个账户 env 文件权限 600，不在日志/聊天回显私钥/API secret。
2. 每个账户创建独立 `ClobClient`，独立 funder/proxy wallet。
3. 执行前逐账户检查 HK、balance、allowance。
4. 从 Gamma 最新市场取 token，不复用旧 token。
5. 下单后每账户分别记录 orderID/tx/hash。
6. 用 data-api positions 按 `proxyWallet/funder` 分账户回读。
7. 跨账户汇总只汇总金额，不合并不同市场份额。

## 推荐命令形态

```bash
python scripts/pm_multi_account.py report --accounts configs/accounts.yaml
python scripts/pm_multi_account.py dry-run --plan /tmp/order_plan.json --accounts configs/accounts.yaml
python scripts/pm_multi_account.py live --plan /tmp/order_plan.json --accounts configs/accounts.yaml --confirm RUN_ID
```

## 账户接入与实盘测试流程

新增账户不要直接 live：

1. 将凭据写入 `/opt/polymarket-worldcup/.env.accounts/<label>.env`，权限 600；`configs/accounts.yaml` 中先 `enabled: false`。
2. 只读解析 `.env`：确认 `POLYMARKET_PRIVATE_KEY/API_KEY/API_SECRET/API_PASSPHRASE/SIGNATURE_TYPE/FUNDER` 都非空。
3. 用私钥本地推导地址，只报告截断地址；若推导地址等于任何曾在 Telegram 明文出现的地址/私钥，阻断 live 并要求轮换。
4. HK guard：确认出口 `country=HK`。
5. 先调用 CLOB `balance-allowance` 验证 API auth、余额、allowance；这一步失败时绝不 `post_order`。
6. 再选一个未来且流动性足够的低风险测试市场，从 Gamma 当前数据取 token，并查 orderbook。
7. 最小实盘测试仅 1 USDC 左右；成交后等待 8-20 秒，用 positions 回读，必要时用 activity 兜底。
8. 只有只读验证 + 1 USDC 测试都通过，才把账户改为 `enabled: true` 并纳入多账户分配。

### 401 Unauthorized/Invalid api key 排查

`balance-allowance` 返回 `401 Unauthorized/Invalid api key` 时，通常表示 API key、secret、passphrase 与当前私钥/funder 不是同一套，或凭据尚未生效。处理方式：

- 不下单、不尝试绕过；
- 让用户在服务器安全文件中重新填同一账号生成的三件套；
- 不要把 account1 的 secret/passphrase 混给 account2；
- 修好后从 `balance-allowance` 重新测起。

## 失败处理

- 某账户余额不足：跳过该账户，不自动超配其他账户，除非主人确认。
- 某账户 allowance 不足：停止该账户并报告。
- 某账户非 HK / 网络异常：停止全部 live，避免部分账户成交后无法对冲。
- data-api 延迟：等待 10-20 秒重查 positions，再用 activity 兜底。
- `401 Unauthorized / Invalid api key`：先不要猜测是混用了主账户。做红acted配置核验：确认脚本只读取该账户 env 路径；列出各字段是否存在、长度、前后缀；比较 account1/account2 的 derived address、funder、api key 前缀是否不同；用 signature_type 0/1/2 + 带/不带 funder 的矩阵测试。若全部 401，则结论是该账户 apiKey/secret/passphrase 不匹配、复制错或已失效，而不是下单 bug。
- 用户上传 Python 下单脚本作为账户来源：只作为 `account*_source_*.py` 保存，权限 600；不要直接执行旧脚本。应抽取/转换到标准 `.env.accounts/accountN.env`，先只读验证 balance/allowance，再做 1 USDC live test。
- 私钥或 API 凭据在 Telegram/聊天中明文出现：标记为泄露，不写入实盘配置，不签名下单；可保存占位/来源文件，但账户保持 `enabled: false`，要求用户通过服务器安全文件或加密通道填入新凭据。
- `401 Unauthorized / Invalid api key`：先证明是否真的读取目标账户 env，不要让用户怀疑混用 account1。输出脱敏核对：env 路径、private key 派生地址、funder、API key 前后缀/长度；再用 signature_type 0/1/2 × 带/不带 funder 做 auth matrix。如果全部 401，结论是该账户 API key/secret/passphrase 不匹配或失效，而不是误用 account1。
- 用户上传 Python 下单脚本作为账户来源时：只作为 `account2_source_*.py` 保存，真正 live 系统仍读取 `.env.accounts/<label>.env`。先从脚本 AST 提取变量名做脱敏盘点，不直接执行来源脚本。
- 私钥或 API key 已在 Telegram 明文出现时：该凭据按泄露处理，不写入正式 live env，不签名下单；仅可保存来源文件或占位模板，并要求换新钱包/新 API 凭据。

- data-api 延迟：等待 10-20 秒重查 positions，再用 activity 兜底。
