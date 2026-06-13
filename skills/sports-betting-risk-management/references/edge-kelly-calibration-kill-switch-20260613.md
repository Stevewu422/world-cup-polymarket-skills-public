# Edge / 1-4 Kelly / Calibration Kill-switch Rules (2026-06-13)

## Summary

This reference centralizes the updated World Cup / Polymarket staking discipline. It is intentionally conservative and can stop live betting when the system fails to prove edge.

## Executable edge gate

1. Estimate model probability `q` before reading the market.
2. Read executable buy price `p` from best ask/sell-one price, including spread.
3. Only execute if:

```text
q - p >= 0.05
```

If not, write `无 edge / 不下`.

## Sizing

Use 1/4 Kelly as the base:

```text
base_fraction = 0.25 * (q - p) / (1 - p)
```

Confidence, injury/lineup uncertainty, liquidity, correlation, and portfolio concentration can only shrink this result. A/B/C bands are sanity caps only.

## Stop rules

- Daily drawdown >= 3%: stop adding new positions.
- Peak-to-trough drawdown >= 8%: cut all open exposure by 50%.
- Rolling 50 bets outside model 95% expected interval: pause and audit.
- Three consecutive wrong bets: review signal only.

## Calibration ledger

Each real-money bet must log:

```text
timestamp, match, market, outcome, q, p, size, notional, closing_price, result, CLV
```

Rolling 50-bet kill-switch:

```text
if model_brier >= market_brier and average_CLV <= 0:
    stop real-money betting
    switch to research / dry-run mode
```

This can intentionally conclude that the system has no edge. That is not a bug.

## Data integrity gate

- Fixture cross-check.
- Polymarket slug/team-code verification.
- Market title/outcome/token mapping.
- Timestamped price; refetch if older than 15 minutes.
- BLOCK live execution if this fails; dry-run only.

## Cross-direction cap

Same-direction exposure across matches (e.g. all favorites or all unders) must stay <= 5% of bankroll unless reduced elsewhere and calibration remains healthy.

## Compliance warning

HK routing, multi-account execution, pooled funds, and participant accounting are compliance-sensitive and require qualified review; they are not just technical optimization tasks.
