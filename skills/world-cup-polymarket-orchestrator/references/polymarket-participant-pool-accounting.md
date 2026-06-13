# Polymarket participant pool accounting

## 触发场景

当主人要求记录“谁参加下注 / 每人本金 / 当前账面总金额 / 每人余额和池子占比 / 某场利润按比例分配”时使用。

## 数据结构

建议在私有目录保存，不进入公开 skill 包：

```text
/root/.hermes/private_tasks/coco/polymarket_betting_participants/
  latest.md
  current_pool_balance_YYYYMMDD.md
  <match>_profit_allocation_YYYYMMDD.md
```

## 计算口径

### 1. 本金合计

```text
principal_total = sum(participant_principal)
```

### 2. 当前净利润

用户给出当前账面总金额时：

```text
current_profit = current_book_value - principal_total
```

### 3. 利润按指定比例分配

如果用户明确说某场/本轮利润分配比例，例如：

```text
粟粟 40%，ray 20%，Steve 40%
```

则：

```text
participant_balance = principal + current_profit * profit_share
```

没有被指定分润的人，本轮默认只保留本金；不要自动按本金比例分利润，除非用户明确说“按本金比例”。

### 4. 池子占比

```text
pool_share_pct = participant_balance / current_book_value * 100
```

金额一般保留 2 位，内部校验可保留 4 位。

## 输出模板

```text
当前账面总金额：9941 USD
本金合计：9864 USD
当前净利润：77 USD
利润分配：粟粟 40%，ray 20%，Steve 40%

- 粟粟：757.80 USD，占池子 7.62%
- ray：315.40 USD，占池子 3.17%
- 假装在伦敦：727.00 USD，占池子 7.31%
- 石桥：624.00 USD，占池子 6.28%
- Steve：7516.80 USD，占池子 75.61%

校验：余额合计 = 当前账面总金额
```

## Pitfalls

- 不要把“参与本金”误当成当前余额；当前余额应等于本金 + 分配利润/亏损。
- 不要在用户已指定分润比例时改成按本金比例。
- USDC/USD 在 Polymarket 记账里通常按 1:1 处理，但输出要沿用用户说法。
- 记录参与人姓名时按用户原文保留，例如“假装在伦敦”。
- 涉及私钥/API 凭据的账户文件不要和参与人账本混放。
