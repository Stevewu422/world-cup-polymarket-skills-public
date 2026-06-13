# Polymarket 足球下注：仓位与份额报告口径

## 触发场景

用户要求对多场比赛、多市场下注，并要求说明“每场多少钱、多少份额”时使用。尤其适用于世界杯/足球的 moneyline + totals/O-U 组合下注。

## 核心纠错

### 1. 金额可以按场合计，份额不能跨市场合计

同一场比赛可能同时买：

- 胜负市场：例如 `Canada win YES`
- 大小球市场：例如 `Canada vs Bosnia Under 2.5`

这两个是不同 condition/asset/token，份额单位不是同一种合约，不得相加。

错误写法：

```text
Canada 本场合计：104.0732 USDC，189.8799 份
```

正确写法：

```text
Canada 本场金额敞口：104.0732 USDC
Canada 胜：55.0988 USDC，103.9600 份，均价 0.53
Canada Under 2.5：48.9744 USDC，85.9199 份，均价 0.57
```

### 2. 区分“本轮新增买入”和“最终总持仓”

用户说：

```text
这轮总共买入 300 USDC / 再下 100 USD / 追加 100
```

应按本轮新增 notional 计算，而不是把最终总持仓补到该金额。

必须报告三层：

```text
已有持仓：执行前该市场 initialValue
本轮新增：本轮订单 makingAmount / notional
最终持仓：执行后该市场 initialValue
```

若误把“本轮新增 300”理解成“最终持仓 300”，需要立刻修正：

```text
本轮应新增 = 用户指定金额 - 本轮已实际新增
按原比例补买剩余金额
```

### 3. 推荐输出格式

```text
本轮新增总额：299.98 USDC

Mexico vs South Africa
本场新增金额敞口：122.44 USDC
- Mexico 胜：67.35 USDC，96.21 份，均价 0.70
- Mexico Under 2.5：55.10 USDC，96.66 份，均价 0.57

Korea Republic vs Czechia
本场新增金额敞口：73.47 USDC
- Korea 胜：30.61 USDC，82.73 份，均价 0.37
- Korea Under 2.5：42.86 USDC，72.64 份，均价 0.59

Canada vs Bosnia-Herzegovina
本场新增金额敞口：104.07 USDC
- Canada 胜：55.10 USDC，103.96 份，均价 0.53
- Canada Under 2.5：48.97 USDC，85.92 份，均价 0.57
```

## Reporting checklist

Before replying after execution:

1. Confirm whether numbers are `本轮新增` or `最终总持仓`.
2. Aggregate only money by match.
3. Never aggregate shares across markets.
4. For each independent market, report: amount, shares, avg/limit price, status/order id if execution occurred.
5. If adding to previous holdings, include old/new/delta when the user asks for audit detail.
