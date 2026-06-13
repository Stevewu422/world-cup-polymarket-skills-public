# Polymarket live execution + proportional staking notes

Use this reference when the user asks the agent to place football/World Cup bets on Polymarket.

## Execution safety

- Real orders must run from the user-approved Hong Kong execution host, not the agent's default host, when the user has specified regional execution constraints.
- Before any live order, verify:
  - egress region is HK;
  - market is active and not closed;
  - `clobTokenId` is current and orderbook exists;
  - balance and allowance are sufficient;
  - token maps to the intended outcome (Yes/No ordering from `outcomes` and `clobTokenIds`).
- Use dry-run/orderbook checks before live execution.
- Keep private keys/API secrets out of chat. If keys were sent in chat, warn that they should be treated as exposed.

## Getting valid clobTokenIds

- Use Polymarket Gamma API market/event data; valid token IDs are in `markets[].clobTokenIds`.
- The index maps to `markets[].outcomes`: `outcomes[0]` -> `clobTokenIds[0]`, etc.
- A token is not safe to trade until orderbook lookup succeeds. `invalid token id` or `No orderbook exists` means do not place the order.

## Proportional staking

When the user says “1份=10USD” or similar, treat that as a base unit, not as a mandatory size for every market.

Recommended output per pick:

```text
信心等级：A/B/C
价值等级：高/中/低
目标份数：X份（约 X*base_amount）
现有持仓：Y份
本次补单：max(0, X-Y)份
仓位理由：为什么比其他场更大/更小
```

Guidelines:

- B-/high variance: 0.3-0.5份.
- B: 0.5-0.8份.
- B+: 0.8-1.0份.
- A-/A: 1.0-1.5份.
- Single match total exposure across correlated markets should not exceed 2份 unless the user explicitly overrides.
- Totals/大小球 are often secondary unless the totals signal is stronger than the side.
- If a prior test order already created exposure, only top up to the target size; do not add another full unit.

## Reporting after execution

For each filled order, report briefly:

- match and market;
- side/outcome;
- limit price and size;
- notional amount;
- status (`matched`/failed/open);
- order id and transaction hash when available;
- before/after balance if checked.

Avoid guaranteed-win language; phrase as probability/value and risk-controlled allocation.