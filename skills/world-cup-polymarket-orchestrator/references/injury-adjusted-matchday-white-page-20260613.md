# Matchday injury-adjusted analysis + public white long-image page (2026-06-13)

## Durable lessons

1. Late injury updates can reverse totals logic. Example: Brazil vs Morocco moved from `Brazil win + Under 2.5` toward `Brazil win hold + Under downgraded + BTTS/Over risk` because Brazil's right-back weakness faced Hakimi while Morocco's center-back injuries increased Brazil scoring paths.
2. A clean injury sheet does not equal value. Qatar vs Switzerland had low injury uncertainty, but Switzerland ML and -1.5 were already priced high; hold existing exposure, avoid -2.5 chase.
3. Core-player availability can strengthen ML but weaken spreads/totals. Haiti vs Scotland: McTominay available supported Scotland ML, but Bellegarde uncertainty and first-match low-block risk meant Scotland -1.5 and Over 2.5 should be downgraded.
4. For Australia vs Türkiye, one named playmaker (`Çalhanoğlu`) became the trigger: starts -> tiny Türkiye ML allowed; absent -> abandon Türkiye ML and prefer Under/Australia resistance. Türkiye -1.5 remained no-touch.
5. When the user asks to update a share page, verify both URL content and a screenshot. If they reject numeric IP, use a verified hostname such as `sslip.io`. If they ask for a white-image version, publish a simple page embedding the verified white PNG instead of a dark card dashboard.

## Output pattern that worked

- Make a white 1080px wide long PNG with four cards, each card containing:
  - matchup/time
  - current Polymarket price snapshot
  - injury/lineup variable
  - direction/action
  - score lean
  - explicit no-touch / downgrade instruction
- Publish an HTML wrapper that embeds the PNG under `/assets/...png?v=<timestamp>`.
- Verify:
  - page HTTP 200
  - expected title/body strings
  - image asset HTTP 200 and nonzero size
  - screenshot renders the white long image

## Pitfall to avoid

Do not keep old secondary recommendations after the user brings stronger injury data. State the downgrade in plain language and, if there is already exposure, provide a reduction plan before suggesting any new hedge.
