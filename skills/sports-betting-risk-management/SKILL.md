---
name: sports-betting-risk-management
description: Responsible sports-betting analysis workflow for football/soccer tournaments and match-by-match betting plans. Use when the user asks for 世界杯赌球策略, 足球下注策略, betting picks, bankroll management, odds/value analysis, group-stage match plans, or gambling risk control. This skill emphasizes fixture/odds verification, conditional recommendations, bankroll limits, and no guaranteed-win claims.
version: 2.0.0 (edge- and calibration-coupled patch)
---

# Sports Betting Risk Management

> 【v2 摘要】保留全部 injury/live/consent 纪律不变；仅在三处动刀：
> ①仓位由 edge 推导（1/4 凯利），信心只作收缩；②停手用回撤/置信区间，弃"连黑3场"硬停；
> ③新增校准账本 + Kill-switch，并加跨场同方向上限与最低 edge 门槛。
> 目标是保本与诚实，不是提高胜率。

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

【v2】Minimum edge + size by edge. Stating the value gap is not enough — gate on it.
Only treat a pick as executable when
value gap = model_prob − ask_price_implied_prob ≥ 5 percentage points,
using the ask / buy price (which already includes Polymarket's spread = your real cost),
not the mid. A pick whose edge vanishes once you use the ask price is a pass.
And size by the magnitude of the edge (fractional Kelly, workflow §4), not by the confidence grade alone.

【v2】Separate **HOLD** from **ADD**. Existing positions may remain acceptable even when the current ask price no longer offers ≥5pp edge. In reforecast or late-match updates, explicitly classify each leg as `HOLD / NO ADD / ADD CANDIDATE / TRIM / EXIT`. Never convert a qualitative “方向仍可” into a fresh add unless the current buy-price edge gate passes.

## Required workflow

For Polymarket live execution and proportional-staking details, see references/polymarket-live-execution-and-proportional-staking.md.

For late injury/lineup-driven re-rating and concrete downgrade actions, see references/worldcup-injury-lineup-adjustment-20260613.md.

Official lineup confirmation timing: public confirmed XI normally appears around T-60 minutes, while team sheets may be submitted earlier around T-75 to T-90. Start watching at T-70, but treat leaks/predicted XI as `PROVISIONAL`; only FIFA/team official/Sofascore/FotMob confirmed XI upgrades lineup status to `CONFIRMED`. If T-55 still lacks official XI, apply lineup multiplier <=0.5 and do not upgrade stakes.

1. **Verify fixtures first**
   - Check current schedule, teams, kickoff time, and timezone before giving a plan.
   - Convert to the user's relevant timezone when useful, especially Beijing time / Bangkok time.
   - **【v2】Slug integrity check.** Parse each Polymarket slug's team codes and confirm they match the fixture (e.g. Germany vs Curaçao must be `ger-cuw`-like; a `ger-kor` slug = Korea, not in the group → `SLUG_MISMATCH`, drop the link, never bet on it). Timestamp every price; if older than 15 min, re-pull.

2. **Ask for or state odds dependency**
   - If no live odds are provided, give conditional thresholds instead of absolute picks.
   - Example: “If Mexico DNB is above X, small stake; if 1x2 price is too low, pass.”
   - Convert odds/Polymarket price into implied probability when possible, then compare with the model probability. Do not call a pick “value” without this comparison.
   - **【v2】Use the ask/buy price for the implied probability**, not the mid — the spread is your cost. Estimate model probability *before* reading the market price to avoid anchoring; if model ≈ market, flag possible anchoring and recheck.

3. **Classify bet types**
   - Safer: Draw No Bet, Asian handicap 0 / +0.25 / +0.5, underdog with spread, small-ball where tactical conditions fit.
   - Higher variance: outright underdogs, correct score, half/full-time, large accumulators.
   - Usually avoid as core strategy: correct score heavy staking, emotional chase betting, large parlays.

4. **Set bankroll limits and proportional stakes before picks**
   - Define total bankroll as 100 units.
   - Treat “1份” as the user's current base stake (for this user often 10 USDC), but **do not mechanically bet 1份 on every pick**.
   - **【v2】Size by edge, not by grade.** Base the stake on quarter-Kelly:
     ```
     f_kelly = (q − p) / (1 − p)        # q = model prob, p = ask/buy price
     base    = 0.25 × f_kelly
     stake   = base × conf_mult × lineup_mult
       conf_mult:   A=1.0  B=0.7  C=0.0
       lineup_mult: confirmed=1.0  unconfirmed=0.5
     ```
     Confidence and lineup only **shrink** the edge-derived size; they never inflate it.
   - **【v2】The old grade→份 ranges are kept only as sanity caps, not as the sizing rule**:
     B-/high variance ≤0.5份; B ≤0.8份; B+ ≤1.0份; A-/A ≤1.5份.
     A B-grade pick with a tiny edge should be *smaller* than its range; with a large edge it may reach the cap.
   - Maximum total risk on one match across all correlated markets (1X2 + totals + props) is 2份.
   - **【v2】Cross-match directional cap.** Tag each bet (CHALK / UNDER / UNDERDOG / …);
     same-direction exposure summed across matches ≤ 5% of bankroll (≈ 4–5份).
     The per-match 2份 cap does not catch a whole slate that is all chalk / all unders.
   - **【v2】Capital mode.** Once a real bankroll or 250/500/1000-level stake is given, switch to %:
     single match 1–2.5% reasonable, 2.5–5% aggressive (confirm), >5% not advised;
     total open exposure 15–25% → mark “aggressive”, stop new adds, only trim/lock/stop-loss.
   - If a position already exists, only top up to the target stake; do not add a fresh full unit.
   - **【v2】Stops by drawdown and variance, not by streak.**
     - Daily: stop new bets after 5 units / 3% bankroll drawdown for the day. (kept)
     - Peak drawdown: after 8% drawdown from the account peak, halve all sizing until back within 3% of peak.
     - “Is the model broken?” check: over a rolling 50 bets, if realized results fall outside the model's 95% interval (clearly worse than the model implies), pause and investigate.
     - The old “stop after 3 consecutive losses” is retained **only as a soft prompt to recheck, not a hard stop** — 3 losses in a row is normal variance even with positive edge (~9% at 55% win prob).
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

## Calibration ledger & kill-switch (v2 新增)

The whole point of this section: let data, not feeling, decide whether the system is allowed to bet real money.

Maintain — jointly with `world-cup-daily-prediction` — a ledger of every **confirmed** bet:
`date / match / market / model_prob q / ask_price p / result / closing_price / CLV`.

Every rolling 50 bets, compute and record:

- **Brier (model)** = `mean((q − result)^2)`, and **Brier (follow-the-market baseline)** for the same bets.
- **Average CLV** (your entry vs the closing line). A persistently **positive average CLV is the single most reliable evidence of real edge.**

**Kill-switch (hard rule):**
- If `model Brier ≥ baseline Brier` **AND** `average CLV ≤ 0` → set `stake × 0` (research mode only) until the method improves and re-passes the check.
- If the model only marginally beats the baseline (within noise) → apply an extra `× 0.5` caution factor.

Stake sizing in §4 **must consume this signal**. This is what stops you (and any pooled participants) from putting real money on a system that has not been shown to work.

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
- **【v2】Do not mark a pick executable when the value gap is < 5pp (computed on the ask price).**
- **【v2】Do not hard-stop on a 3-loss streak alone; use drawdown / confidence-interval stops — a streak is usually variance, not a broken model.**
- **【v2】Do not output a Polymarket link that failed the slug team-code check.**

## Compact template

```text
结论：本场只小注/观望/可打，不重仓。

比赛：A vs B
时间：北京时间 YYYY-MM-DD HH:MM

主方向：...
副方向：...
模型概率 / 市场隐含(买入价) / 价值差：...%/...%/...pp   （【v2】<5pp 即 pass）
仓位：按 edge 推导（1/4 凯利，信心/首发收缩，封顶 2份/场）
赔率阈值：...
不碰：...
赛前复核：首发、盘口变化、大小球变化、伤停、动机。
停手机制：【v2】单日回撤3% / 峰值回撤8%减半 / 50注校准打不过基准则Kill-switch；不追单。
```
