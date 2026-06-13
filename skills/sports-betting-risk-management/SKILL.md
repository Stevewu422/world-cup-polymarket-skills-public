---
name: sports-betting-risk-management
description: Responsible sports-betting analysis workflow for football/soccer tournaments and match-by-match betting plans. Use when the user asks for 世界杯赌球策略, 足球下注策略, betting picks, bankroll management, odds/value analysis, group-stage match plans, or gambling risk control. This skill emphasizes fixture/odds verification, conditional recommendations, bankroll limits, and no guaranteed-win claims.
---

# Sports Betting Risk Management

## Trigger

Use this skill when the user asks for:

- 世界杯/欧洲杯/足球/体育 betting or 赌球策略
- 从第一场开始制定下注计划
- 胜平负、让球、大小球、串关、正确比分建议
- 赔率价值、盘口变化、资金管理、止损纪律

## Core principle

Do **not** frame betting as prediction certainty. Treat it as:

1. information verification,
2. odds/value comparison,
3. bankroll discipline,
4. conditional execution based on lineup, price, and market movement.

Never promise sure wins, stable profit, or risk-free returns.

Separate **prediction probability** from **betting value**. A team can be more likely to win but still be a pass if the market price is too expensive. For every A/B recommendation, state the model probability range, market-implied probability, value gap, and the price at which the pick becomes a pass.

## Required workflow

For Polymarket live execution and proportional-staking details, see `references/polymarket-live-execution-and-proportional-staking.md`.

For late injury/lineup-driven re-rating and concrete downgrade actions, see `references/worldcup-injury-lineup-adjustment-20260613.md`.

1. **Verify fixtures first**
   - Check current schedule, teams, kickoff time, and timezone before giving a plan.
   - Convert to the user's relevant timezone when useful, especially Beijing time / Bangkok time.

2. **Ask for or state odds dependency**
   - If no live odds are provided, give conditional thresholds instead of absolute picks.
   - Example: “If Mexico DNB is above X, small stake; if 1x2 price is too low, pass.”
   - Convert odds/Polymarket price into implied probability when possible, then compare with the model probability. Do not call a pick “value” without this comparison.

3. **Classify bet types**
   - Safer: Draw No Bet, Asian handicap 0 / +0.25 / +0.5, underdog with spread, small-ball where tactical conditions fit.
   - Higher variance: outright underdogs, correct score, half/full-time, large accumulators.
   - Usually avoid as core strategy: correct score heavy staking, emotional chase betting, large parlays.

4. **Set bankroll limits and proportional stakes before picks**
   - Define total bankroll as 100 units.
   - Treat “1份” as the user's current base stake (for this user often 10 USDC), but **do not mechanically bet 1份 on every pick**.
   - Size each pick by confidence + market value: B-/high variance 0.3-0.5份; B 0.5-0.8份; B+ 0.8-1.0份; A-/A 1.0-1.5份.
   - Maximum total risk on one match across all correlated markets (1X2 + totals + props) is 2份.
   - If a position already exists, only top up to the target stake; do not add a fresh full unit.
   - Stop after 3 consecutive losses.
   - Stop for the day after 5 units drawdown.
   - No martingale, no chase, no doubling to recover losses.

5. **Give match-by-match plan**
   - For each match, provide:
     - lean / no-bet direction,
     - preferred market,
     - odds threshold if no live odds,
     - stake size,
     - avoid list.
     - model probability vs market-implied probability,
     - strongest counterargument and downgrade trigger,
     - pass price where the edge disappears.
   - **Execution consent must be market-specific.** If the plan contains multiple correlated legs (win + spread + total), do not treat a general “继续/执行/可以” as approval to place every leg when recent discussion included uncertainty or injury-driven downgrades. Ask for or require a precise execution phrase naming the exact legs, e.g. “执行苏格兰胜，不执行 -1.5/O2.5” or “执行A：两个账户都减半小2.5”.
   - When fresh injury/lineup information arrives after a plan, re-rank the existing legs before any execution: keep/downgrade/exit/add. If a leg is downgraded from main/secondary to observation, explicitly say “不再新增，只保留/减仓” rather than leaving it in the executable bundle.

5b. **Live/in-play adjustment discipline**
   - If the user asks during a live match (e.g. “58分钟 0:1 / 78分钟 1:1 / 27分钟 2:0”), first identify existing positions and current market prices before recommending changes.
   - For Polymarket, use account `positions` for exact assets and CLOB orderbook best bid/ask as the conservative execution value.
   - Distinguish tactical scenario math from audited PnL: live advice can estimate outcomes, but “真实盈亏” must be audited later from actual activity/SELL cashflows plus remaining current value.
   - Do not recommend chasing the original side after adverse game state changes; prefer reducing correlated exposure, locking partial profit, or standing aside.
   - When a totals bet loses its safety margin (e.g. Under 2.5 at 1:1 late or 2:0 early), explicitly reassess and usually reduce exposure unless price is already too poor and remaining time is very short.

6. **Tournament phase logic**
   - First group match: conservative, lower stake, prefer no-loss and unders where justified.
   - Second group match: adjust based on real performance and market overreaction.
   - Third group match: prioritize motivation, qualification math, rotation, and incentive asymmetry.
   - Knockout stage: account for extra time/penalties and distinguish 90-minute markets from qualification markets.

7. **Injury/lineup-driven re-rating and correlated-market discipline**
   - When the user supplies fresh injury/lineup intel, explicitly re-rate each existing recommendation: keep / downgrade / exit / wait for lineup. Do not cling to the earlier thesis.
   - If new injuries undermine a totals thesis (e.g. defender injuries on both sides weaken Under 2.5), downgrade it from “main line” to “protection/observation” and give an exact reduction plan (e.g. sell 1/3, 1/2, or leave only 30%-35%).
   - Separate **main side** from correlated add-ons. Do not execute or recommend adding both a handicap and a total as normal副仓 just because the favorite is stronger; treat -1.5, O/U, BTTS, and correct score as optional correlated risk requiring explicit confirmation per market.
   - If the user questions why a correlated market was executed, acknowledge and correct: preserve the core side if still valid, reduce optional correlated markets first, and report current position sizes before proposing trades.
   - For “wait for lineup” matches, name the single trigger player/position (e.g. a playmaker or fullback) and provide branches: if starts → small side allowed; if out → abandon side, prefer total/underdog resistance or no bet.

## Injury / lineup update adjustment discipline

When fresh injury, illness, rotation, or predicted-lineup information arrives after an initial plan or after positions already exist:

1. **Re-rate the thesis before touching execution.** Explicitly state which markets are upgraded/downgraded and why. Injury news can invert totals logic (e.g. one side loses defenders while the other has an exploitable fullback weakness → Under may downgrade even if a creator is out).
2. **Audit existing positions first.** Read exact holdings, size, avg price, current value, and best bid/ask before recommending changes. Do not discuss “reduce” in abstract if live positions exist.
3. **Downgrade means size reduction, not necessarily reversal.** Default sequence: stop adding → sell 1/3 or 1/2 of the downgraded correlated market → keep a smaller protection/lottery remainder → only hedge with the opposite market after final lineup confirmation.
4. **Do not execute a whole combo from a general recommendation.** For multi-leg football cards, require explicit per-leg authorization before placing/adding secondary markets such as -1.5, Over/Under, BTTS, or correct score. If the user says they handled a leg manually, verify positions/activity and then continue from the new state.
5. **Preserve the main edge when secondary markets weaken.** If injuries strengthen the side/win market but weaken totals/handicap, keep the side as main exposure and downgrade the correlated secondary legs.
6. **Name the trigger player(s).** For lineup-dependent plans, define the exact player/role that changes the plan (e.g. “if Çalhanoğlu starts, tiny Türkiye win is allowed; if not, abandon Türkiye win and prioritize Under/Australia resistance”).

## Injury / lineup revision discipline

When the user brings late injury or predicted-lineup information into a match plan, treat it as a first-class model update, not as commentary on top of the old recommendation.

1. Reclassify each existing market into: **keep as main line**, **downgrade to protection/observation**, **reduce/hedge**, or **do not add**.
2. If injuries attack the original thesis, explicitly name the downgrade. Example: a prior `Under 2.5` should become "protection only" when both teams' defensive injury profiles and flank matchups now support BTTS/Over risk.
3. If lineups are unresolved, give trigger rules tied to 1-3 named players rather than a fixed pick. Example: "If Çalhanoğlu starts, tiny Türkiye ML is allowed; if not, abandon Türkiye ML and prefer Under/Australia resistance."
4. Be conservative with correlated secondary markets. Do not execute or recommend holding both a favorite's `-1.5` and `Over 2.5` as normal secondary exposure after new information says the favorite may struggle to break a low block or the underdog's key creator is uncertain. Downgrade them to small/observation unless confirmed lineups support them.
5. If the user challenges "why did you execute X and Y", acknowledge the execution/approval ambiguity and immediately provide a reduction plan. Do not defend the old thesis.
6. For manual user actions ("I already executed方案A"), verify/record the resulting exposure if tools are available, then continue from the new state; do not assume the prior target remains.

## Live adverse-score adjustment

When the user asks what to do because an existing pick is losing live (e.g. favorite down 0:1):

1. **Do not default to chasing.** Start with “stop adding to the original side” unless live price, dominance, and time remaining clearly justify a controlled top-up.
2. Separate each correlated market already held:
   - Side/win market: often becomes the first stop-loss or hedge candidate.
   - Totals market: reassess by scoreline + minute + match tempo; a 0:1 score can still support Under 2.5 if late/slow, but is risky if early/open.
3. Always ask for or verify the live minute, current price/orderbook, and liquidity before recommending execution.
4. Give conditional branches by time band:
   - Before ~60': if the favored team is not dominating, reduce/hedge the side 50–70%; do not add full fresh exposure.
   - 60–75': side recovery probability decays; prefer partial stop-loss and preserve only the market with a still-valid thesis.
   - After ~75': usually abandon the original win-side chase; focus on protecting remaining recoverable positions.
5. Report current exposed cost by market and, if a pool ledger exists, show affected participant allocation.
6. Keep execution separate from advice: “this is adjustment guidance, not a live order” unless the user explicitly authorizes the trade.

## Output style for this user

- Chinese, conclusion first.
- Directly state “主方向 / 副方向 / 仓位 / 不碰什么”.
- Avoid overclaiming.
- Include a short responsible-gambling warning without moralizing.
- If the user expects voice, also provide a TTS summary in normal task execution.

## Red flags / pitfalls

- Do not say “稳胆/必胜/稳赚”.
- Do not recommend all-in or aggressive recovery after loss.
- Do not recommend high-risk parlays as the main plan.
- Do not give outdated fixtures from memory; verify current schedule.
- Do not treat team reputation as enough; account for lineups, odds, motivation, travel, weather, and market movement.

## Compact template

```text
结论：本场只小注/观望/可打，不重仓。

比赛：A vs B
时间：北京时间 YYYY-MM-DD HH:MM

主方向：...
副方向：...
仓位：... 单位
赔率阈值：...
不碰：...
赛前复核：首发、盘口变化、大小球变化、伤停、动机。
停手机制：连黑3场停；当天亏5单位停；不追单。
```
