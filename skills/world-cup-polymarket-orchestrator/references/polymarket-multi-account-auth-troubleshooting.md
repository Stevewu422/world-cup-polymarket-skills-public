# Polymarket 多账户认证排障与安全测试

## 触发场景

当主人怀疑“是不是用了 account1 的 API 去测 account2”、多账户下单返回 `401 Unauthorized / Invalid api key`、或需要验证 account2 是否真正独立读取时使用。

## 核心原则

- 不回显 private key、API secret、passphrase；只报告字段是否存在、长度、前后缀。
- account1 与 account2 必须分别从独立 env 文件读取：
  - account1: `/opt/polymarket-worldcup/.env`
  - account2: `/opt/polymarket-worldcup/.env.accounts/account2.env`
- 实盘测试脚本必须显式写死或传入目标 env 路径，避免 `load_dotenv()` 默认加载 `.env`。
- 如果 private key 曾在 Telegram/聊天中明文出现，默认视为泄露，不用于 live signing；只允许只读/结构核验，除非主人已换新私钥并通过服务器安全文件填写。

## 排障顺序

### 1. 对比 account1/account2 摘要

只输出脱敏摘要：

```text
path
POLYMARKET_PRIVATE_KEY present/len/prefix/suffix
derived_address
POLYMARKET_API_KEY present/len/prefix/suffix
POLYMARKET_API_SECRET present/len/prefix/suffix
POLYMARKET_API_PASSPHRASE present/len/prefix/suffix
POLYMARKET_SIGNATURE_TYPE
POLYMARKET_FUNDER redacted
```

判断标准：

- account1/account2 的 private key prefix、API key prefix、funder 应明显不同。
- `derived_address` 与 `POLYMARKET_FUNDER` 可以不同：derived address 是私钥地址，funder 常是 Polymarket proxy wallet/relayer 地址。

### 2. 核验测试脚本实际读取路径

检查脚本是否包含：

```python
ENV = Path('.env.accounts/account2.env')
```

同时确认没有：

```python
load_dotenv()
load_dotenv('.env')
```

如果脚本默认 `load_dotenv()`，要改成显式读取 account2 env 或用 `dotenv_values(path)`，否则容易误用 account1。

### 3. 做 auth matrix 排除 signature/funder 误配置

用 account2 的同一套 API key/secret/passphrase，遍历：

```text
signature_type = 0/1/2
use_funder = false/true
```

每种组合只调用 `balance-allowance`，不下单。

解释：

- 六种组合全部 `401 Unauthorized / Invalid api key`：通常不是 account1/account2 混用，也不是 signature_type/funder 小错，而是 API key/secret/passphrase 不匹配、复制错、过期、被禁用，或不是 CLOB API 凭据。
- 只有某个 signature/funder 组合通过：更新 account2.env 的 `POLYMARKET_SIGNATURE_TYPE` 和 funder 使用方式，再做 1 USDC live test。

### 4. 图片/API key 截图核对注意

截图中 apiKey/secret/passphrase 必须是同一次生成的一整套。常见问题：

- Telegram OCR/视觉摘要无法可靠读取完整密钥；不能靠截图摘要改 env。
- 用户文字里给过旧 API key，服务器 env 里是另一套 key，容易混用。
- 如果页面提示“之后看不到了”，必须当场安全复制到服务器 env；后续无法恢复 secret/passphrase，只能重新生成。

## 1 USDC live test 安全门

只有当以下全部满足才允许实盘小额测试：

1. HK 出口 OK。
2. account2.env 字段齐全。
3. private key 未在聊天中泄露，或已确认是新换私钥。
4. `balance-allowance` 认证通过。
5. balance >= 2 USDC，留 1 USDC 测试 + 缓冲。
6. 从 Gamma 最新 market 获取 token，orderbook 有 ask。
7. 下单 notional 约 1 USDC，并记录 orderID/tx。
8. 下单后等待 8-20 秒，用 positions + activity 复核。

## 报告模板

```text
不是 account1 混用：
- account1 env: .env，API key 前缀 ..., funder ...
- account2 env: .env.accounts/account2.env，API key 前缀 ..., funder ...
- 测试脚本：loads_account2_env=true, loads_dotenv_default=false

认证结果：
- signature/funder matrix: 全部 401 / 某组合通过
- 结论：API 凭据不匹配 / signature_type 修正为 X / 可进入 1 USDC 测试
```

## 禁止

- 不要把用户聊天里发过的私钥写入正式 account2.env 后执行 live。
- 不要把截图里的完整 key/secret/passphrase 发回聊天。
- 不要在 auth 失败时继续尝试 post_order。
