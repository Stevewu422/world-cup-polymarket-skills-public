# Pool ledger and participant allocation for Polymarket betting

## Trigger

Use when the user asks to record who participated in a Polymarket betting pool, update each participant's balance/capital, allocate realized match profit, or split current account positions across people by pool share.

## Core distinction

Keep three ledgers separate:

1. **Pool capital ledger**: each person's balance and share of the pool.
2. **Realized profit allocation**: closed-match profit split by explicitly stated percentages.
3. **Open betting exposure allocation**: current account positions split by pool share, not by realized-profit split unless the user says so.

Do not mix these: a one-match profit allocation can change balances, then those updated balances drive the proportional split of later open positions.

## Workflow

1. Record the user's participant inputs exactly: name + principal/balance + any transfer between participants.
2. If a prior match has realized PnL, apply the user's stated profit percentages before computing current pool shares.
3. If the user corrects a principal or transfer, recompute from the corrected base instead of patching the old total by hand.
4. Compute:
   - `person_balance`
   - `pool_total = sum(person_balance)`
   - `pool_share = person_balance / pool_total`
5. Query current account positions and aggregate only **money/cost** by match. Never add shares/size across different markets.
6. Allocate each match's cost to each person:
   - `person_match_bet = match_cost * pool_share`
   - `person_total_open_bet = total_open_cost * pool_share`
7. Save a concise audit record under the private task directory when requested.

## Output template

```text
口径：
- 账面总额：...
- 已实现利润：...
- 利润分配：...
- 本金/转让修正：...

每个人余额和占比：
- A：余额 X，占比 Y%
- B：余额 X，占比 Y%

account 当前持仓成本：
- Match 1：...
- Match 2：...
- 总成本：...

按池子占比分摊 betting：
A：
- Match 1：...
- Match 2：...
- 合计：...
```

## Pitfalls

- Do not treat the displayed book total as authoritative if exact recomputation gives a small rounding difference; state both if needed (e.g. "9940.9122, approximately 9941").
- Do not allocate open positions by earlier match profit split unless the user explicitly says the same split applies to current exposure.
- Do not merge `moneyline size` with `totals size`; only USDC cost/current value can be aggregated across markets.
- Participant/pool records are task ledgers, not durable memory facts; save them in the private task directory, not memory.
