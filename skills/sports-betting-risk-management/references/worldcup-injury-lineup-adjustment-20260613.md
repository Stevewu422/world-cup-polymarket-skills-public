# World Cup injury/lineup adjustment pattern — 2026-06-13

Use this reference when a user supplies late injury/lineup intelligence that contradicts or weakens an earlier football betting thesis.

## Core lesson

Do not treat a prior pre-match plan as locked after new injury information arrives. Re-score the thesis by market and by correlation, then give a **concrete adjustment**: hold, reduce 1/3, reduce 1/2, close, or wait for lineups. Avoid vague labels like “降级” without a stake action.

## User preference observed

- The user wants explicit authorization before live execution, especially when multiple correlated markets are involved.
- If the user asks why a market was executed, acknowledge whether the execution was too aggressive; do not defend a stale plan.
- For Polymarket, always read positions + recent activity after manual or agent execution before summarizing.
- For every text answer in this betting context, also provide TTS.

## Decision pattern

1. Re-verify current market prices/orderbook for the exact tokens.
2. Re-read current positions and recent activity for all affected accounts.
3. Map the new injury facts to market implications:
   - Missing creator/finisher: lowers favorite scoring efficiency.
   - Missing CB/FB: raises opponent scoring or BTTS/Over probability.
   - Defensive low block + injured creators: supports Under and weakens favorite handicap.
   - Healthy key midfielder/creator: strengthens win market more than handicap unless opponent defense is clearly broken.
4. Convert implication into action:
   - Thesis still valid: hold; no top-up unless explicitly requested.
   - Thesis weakened but not reversed: reduce 1/3–1/2, keep remainder as protection.
   - Thesis invalidated: close or hedge, but only after user approval.
   - Lineup-dependent: wait for official XI; give if/then branches.
5. Report in this shape:
   - Current position by account
   - Current bid/ask
   - Recommended action and exact size/cash estimate
   - Trigger for further action at lineups

## Session examples

### Brazil vs Morocco

Late injury info: Brazil missing Neymar and right-back Wesley; Morocco missing CB Aguerd but has attacking line intact, Hakimi attacks Brazil’s patched right side. This weakens Under 2.5 and supports BTTS/Over, while Brazil win remains acceptable.

Good adjustment:
- Brazil win: hold.
- Under 2.5: stop adding; reduce about 50% while still slightly profitable; keep half as protection.
- Wait for Mazraoui and Brazil RB before adding BTTS Yes / Over 2.5.

### Switzerland vs Qatar

Injuries clean on both sides. Switzerland advantage remains, but Swiss win price is high and -1.5 is not a “safe” bet because Qatar can sit deep.

Good adjustment:
- Switzerland win + -1.5: hold existing exposure.
- Do not chase win at rich price.
- Do not add Switzerland -2.5.
- Only lineup variable: Embolo vs Amdouni / young attackers.

### Haiti vs Scotland

McTominay confirmed fit strengthens Scotland win. Gilmour out is manageable. Haiti Bellegarde status conflicts across sources; if absent, Haiti creativity drops. This supports Scotland win but weakens Over 2.5 and -1.5 as core markets.

Good adjustment:
- Scotland win: hold as main exposure.
- Scotland -1.5: reduce to small/profit-only exposure.
- Over 2.5: reduce 1/3–1/2 or wait for Bellegarde official XI.

### Australia vs Türkiye

Australia full strength and compact low block. Türkiye’s key creators/organizers (Çalhanoğlu, Kenan Yıldız, Ferdi Kadıoğlu) are lineup-dependent; Arda Güler fit. This makes Türkiye win thin and Türkiye -1.5 a bad core bet.

Good adjustment:
- No early Türkiye win unless Çalhanoğlu starts.
- Main lean: Under 2.5.
- If Çalhanoğlu out: abandon Türkiye win; consider Australia resistance/draw direction.
- Never use Türkiye -1.5 as main exposure in this setup.
