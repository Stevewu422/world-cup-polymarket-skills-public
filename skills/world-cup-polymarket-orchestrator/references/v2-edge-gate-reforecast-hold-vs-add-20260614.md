# v2 edge-gate reforecast: separate HOLD from ADD

Use this reference when the user asks to “重新预判 / reforecast / 接下来几场怎么下” after positions already exist.

## Durable lesson

Under v2 risk rules, a direction can remain a valid **hold** while being a **no-add**.

Do not treat “still like the side” as permission to add. Reforecast each open match in two layers:

1. **Position management:** keep / trim / exit existing exposure.
2. **New execution gate:** add only if `q - ask_price >= 5pp` after using the executable buy/ask price.

If the edge gate fails, write explicitly:

```text
方向仍可持有，但无新增 edge / 不加仓。
```

## Required reforecast sequence

1. Re-pull fixture status from a reliable schedule source.
2. Re-pull Polymarket Gamma event/More Markets and CLOB orderbook best bid/ask.
3. Check slug/team-code integrity and mark `FIXTURE ✓ / SLUG ✓ / PRICE(ts=...) ✓`.
4. Estimate model probability `q` before reading/anchoring on price when practical; if updating from a prior estimate, state that this is a re-estimate.
5. Compare against **ask/buy price** `p`, not midpoint.
6. For each market, output: `q / ask p / edge / action`.
7. Separate actions:
   - `HOLD`: existing position still acceptable.
   - `NO ADD`: edge < 5pp.
   - `ADD CANDIDATE`: edge >= 5pp and calibration/data gates pass.
   - `TRIM/EXIT`: thesis weakened or price/lineup invalidates prior position.

## Fallback when web search is unavailable

Do not stop. Use deterministic only-read probes already documented in `live-fixture-market-api-probes.md`:

- ESPN scoreboard for schedule/status.
- Polymarket Gamma `/events?slug=...` and keyset More Markets for event/market discovery.
- CLOB `/book?token_id=...` for executable bid/ask.

Clearly label unavailable injury/news search as:

```text
伤停/首发实时 web 搜索不可用；按待确认处理，临场必须复核。
```

Do not promote a pick because injury data is missing; missing lineup data should only shrink confidence/size.

## Example phrasing

```text
瑞士胜 ask=0.82，模型 q≈0.84，edge≈+2pp：已有仓可持有，但低于 5pp 门槛，不再加。
巴西胜 ask=0.59，模型 q≈0.63，edge≈+4pp：方向仍可，v2 下不算可执行新增。
澳大利亚 vs 土耳其：土耳其胜 ask=0.58，q≈0.53–0.55，无 edge，观望。
```

## Pitfall

Old behavior: after a favorable qualitative reforecast, add 50% to the same match.

v2 behavior: favorable qualitative view is insufficient. Add only when the quantitative edge gate, data integrity gate, and calibration gate all pass.
