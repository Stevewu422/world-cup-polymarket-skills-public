# Injury-driven lineup reassessment for World Cup betting

Session lesson from 2026-06-13 Brazil vs Morocco and Qatar vs Switzerland.

## Trigger

Use when the user supplies fresh injury/lineup intelligence after an earlier betting plan already exists, especially close to kickoff.

## Core pattern

Do not merely append the injury news. Re-run the market thesis:

1. Identify **which line** the injury affects:
   - attacking creation / finishing;
   - fullback or wing channel;
   - center-back depth / aerial defense;
   - goalkeeper certainty;
   - coaching/system uncertainty.
2. Compare both teams' injury locations against each other.
3. Reclassify existing markets:
   - promote / keep / downgrade / cancel;
   - do not keep a prior totals thesis if the new injury map contradicts it.
4. If existing positions already exist, recommend **position management**, not just new picks:
   - hold;
   - stop adding;
   - sell 33% / 50% / 66%;
   - hedge with BTTS Yes / Over 2.5 / opposite totals only after price check.
5. Wait for final lineups when the decisive variable is still unresolved.

## Brazil vs Morocco example

Fresh information:

- Brazil: Neymar out; Wesley out at right back; Alisson available; Gabriel match sharpness/chemistry uncertain; Brazil right-back slot likely Danilo or Edson cover.
- Morocco: Aguerd out and Saiss retired, weakening CB depth; Mazraoui doubtful; attack mostly intact with Hakimi and Brahim Diaz.

Interpretation:

- Neymar out pulls slightly toward lower Brazil chance creation.
- Brazil right-back weakness vs Hakimi pulls toward Morocco chance creation.
- Morocco CB depletion pulls toward Brazil scoring.
- Net effect: downgrade `Under 2.5` from main thesis to protection/secondary; raise interest in `BTTS Yes` or `Over 2.5`, but do not blindly flip all exposure because Neymar absence still creates a genuine under case.

Position-management template if already holding Under 2.5:

```text
Current Under 2.5 bid/ask: X/Y
Existing cost/size: ...
Recommendation: sell 50% now to downgrade from main thesis to protection; keep 50% until lineup.
If final lineup confirms the attacking-risk scenario, reduce remaining under to 30%-35% of original and optionally buy small BTTS Yes / Over 2.5.
If final lineup weakens the attacking-risk scenario, keep the remaining under and avoid fresh hedge.
```

## Qatar vs Switzerland example

Fresh information:

- Switzerland: no injury issue; key old guard retired (Shaqiri, Sommer, Schar), but Akanji, Ricardo Rodriguez, Xhaka, Freuler, Widmer remain; forward choice Embolo vs Amdouni is main variable.
- Qatar: healthy squad, but poor preparation and form; Afif is the main counter threat.

Interpretation:

- Clean injuries reduce last-minute absence risk and support the Switzerland superiority thesis.
- However, Switzerland's market price can already be full; clean health does not automatically create value.
- `Switzerland -1.5` can stand as a secondary position, but avoid chasing `-2.5` because Switzerland are control-oriented and Qatar may defend deep.

## Haiti vs Scotland example

Fresh information:

- Scotland: McTominay illness concern cleared; Gilmour out but replaceable; McKenna minor knock likely OK; goalkeeper choice Gunn vs Craig Gordon is not thesis-changing.
- Haiti: most sources say healthy, but FotMob-style injury tables conflicted on Bellegarde / Frantzdy Pierrot. Bellegarde is the key creative-quality variable.

Interpretation:

- McTominay available strengthens `Scotland win` as the main side.
- It does **not** automatically strengthen `Scotland -1.5` because World Cup opening psychology, low block, and Scotland finishing volatility make a one-goal win plausible.
- `Over 2.5` should not be a main secondary while Bellegarde status is unresolved; if Bellegarde is absent, Haiti creation drops; if he starts, Scotland -1.5 becomes riskier even if Over/BTTS improves.

Position-management template when secondary legs were already bought too aggressively:

```text
Keep Scotland win as main exposure.
Downgrade Scotland -1.5: sell about 50% or stop adding.
Downgrade Over 2.5: sell 33%-50% unless Bellegarde starts and match tempo supports it.
Wait for Haiti XI before any hedge/new totals bet.
```

## Australia vs Türkiye example

Fresh information:

- Australia: full squad, Harry Souttar recovered, likely 5-4-1 / compact 3-4-3 low block.
- Türkiye: Arda Güler healthy, but Çalhanoğlu, Kenan Yıldız, and Ferdi Kadıoğlu were listed as game-time decisions; Çalhanoğlu is the decisive plan trigger.

Interpretation:

- Full-strength Australia plus Türkiye creative-core injury risk downgrades `Türkiye win` from normal side to wait/very small conditional side.
- `Türkiye -1.5` should be cancelled/avoid by default.
- `Under 2.5` becomes the cleaner pre-lineup lean because Australia can slow the game and Türkiye may lack chance creation.

Conditional template:

```text
If Çalhanoğlu starts and appears fit: Under 2.5 main; Türkiye win only tiny/optional; no -1.5.
If Çalhanoğlu does not start: abandon Türkiye win; prioritize Under 2.5 and consider Australia resistance/draw direction.
If Çalhanoğlu + Yıldız + Ferdi all miss/bench: no Türkiye side, no handicap; only small under/resistance exposure.
```

## Execution pitfall from this session

The user questioned why both `Scotland -1.5` and `Over 2.5` had been executed together. Lesson: a multi-leg recommendation is not blanket live authorization. For World Cup cards, especially when late injury information is still pending, execute only the specific leg(s) the user explicitly authorizes. If secondary legs were entered before new injury data, immediately audit positions and propose reduction, rather than defending the old plan.

## Output discipline

Use explicit words:

- `升级`: new information strengthens prior thesis.
- `保留`: still valid, no new money necessarily.
- `降级`: stop adding and reduce size if already oversized.
- `取消`: thesis invalidated; exit or hedge when price/liquidity allows.

Always separate:

- tactical reasoning;
- current price/orderbook;
- existing position size/cost;
- recommended action.
