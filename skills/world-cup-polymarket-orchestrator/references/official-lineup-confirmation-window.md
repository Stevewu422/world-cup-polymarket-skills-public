# Official Lineup Confirmation Window

Use this reference for World Cup / football matchday reforecasting.

## Timing rule

There are two different windows:

- Team sheets are usually submitted to match officials around T-75 to T-90 minutes.
- Public official confirmed lineups are normally released around T-60 minutes.

Therefore, the practical agent workflow is:

1. Start the pre-lineup watch at T-70 minutes.
2. Poll official and fast aggregator sources.
3. Treat predicted/leaked lineups as `PROVISIONAL`, never `CONFIRMED`.
4. Upgrade to `CONFIRMED` only after official XI appears or a reliable aggregator clearly marks confirmed XI.
5. Immediately re-estimate model probability `q`, re-run the price/slug/data gates, and modify recommendations.

## Recommended sources by priority

### One-page aggregator to watch first

1. RotoWire World Cup lineups page: `https://www.rotowire.com/soccer/lineups.php?league=WOC`
   - Verified reachable from this host: HTTP 200, title `World Cup Lineups: Predicted & Confirmed Starting XI for Every Match | RotoWire`.
   - One URL covers all World Cup matches.
   - Before release it can show `lineup has not been posted yet`.
   - When available it fills predicted/confirmed XI and marks OUT/QUES.
   - Best target for cron/manual polling because the status is clear and all matches are in one page.

### Fast push / lineup tabs

2. Sofascore World Cup page: `https://www.sofascore.com/football/tournament/world/world-championship/16`
   - Open match page → `Lineups` tab.
   - Confirmed XI normally appears around T-60.
   - App push can be nearly simultaneous with official release.
3. FotMob match pages: use `fotmob.com` search/open the specific fixture.
   - Predicted XI switches to confirmed XI when available.
   - App push is useful as a second trigger.
4. Flashscore match pages as a fast secondary check.

### Prediction + confirmation comparison

5. lineups.com World Cup lineups: `https://www.lineups.com/world-cup/starting-lineups/`
   - Shows predicted and confirmed lineups by match.
6. SportsGambler FIFA World Cup lineups: `https://www.sportsgambler.com/lineups/football/fifa-world-cup/`
   - Usually posts lineups around one hour before kickoff and can include bench lists.
7. RotoWire / lineups.com / WhoScored / ESPN can be used for predicted-vs-confirmed comparison; never treat their prediction as final confirmation unless explicitly marked confirmed.

### Most authoritative / fastest when available

8. FIFA App / FIFA.com match centre.
9. Both teams' official social accounts / official websites.
   - Team official lineup graphics may appear a few minutes before aggregators.
   - If official team/FIFA source conflicts with aggregator, official source wins.

### Single-match deep pages

- Sofascore single-match pages/articles can be used for detail, e.g. Brazil vs Morocco search result/page.
- Use single-match pages for changed player lists, tactical notes, and confirmation cross-checks, but keep RotoWire one-page as the main cron polling URL.

## Execution discipline

- Before official/confirmed XI: lineup status = `PROVISIONAL`; lineup multiplier <= 0.5; no upgrade based on leaked/predicted XI.
- After confirmed XI: lineup status = `CONFIRMED`; recompute q and edge using the latest ask price.
- If official XI is still missing at T-55, write `LINEUP_NOT_CONFIRMED`, do not upgrade stake; only reduce or keep.
- Because confirmed XI appears around T-60, there is usually only a 30-40 minute execution window. Cron should fire around T-70 to get ready and then act when the confirmed XI appears.

## Polling pattern

For each T-70 cron:

1. Fetch RotoWire WOC page.
2. Locate the target fixture and status.
3. If the page still says `lineup has not been posted yet`, mark `PROVISIONAL` and keep polling/checking official/FotMob/Sofascore manually if needed.
4. If lineups are populated, cross-check against FIFA/team official/Sofascore/FotMob when possible.
5. Then re-run forecast: new q, latest ask p, edge, and action.

## Output requirement

For each reforecast, include:

```text
Lineup status: PROVISIONAL / CONFIRMED / LINEUP_NOT_CONFIRMED
Source: RotoWire / FIFA / team official / Sofascore / FotMob / other
Timestamp: ...
Changed players: ...
New q / ask p / edge / v2 action: ...
Recommendation change: upgrade / keep / downgrade / cancel
```
