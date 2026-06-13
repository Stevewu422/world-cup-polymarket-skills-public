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
4. Upgrade to `CONFIRMED` only after official XI appears.
5. Immediately re-estimate model probability `q`, re-run edge gate, and modify recommendations.

## Recommended sources by priority

1. FIFA App / FIFA.com match centre.
2. Both teams' official social accounts / official websites.
3. Sofascore and FotMob lineup push updates.
4. Flashscore.
5. RotoWire World Cup lineups, lineups.com, WhoScored, ESPN as secondary confirmation / predicted-vs-confirmed comparison.

## Execution discipline

- Before official XI: lineup status = `PROVISIONAL`; lineup multiplier <= 0.5; no upgrade based on leaked/predicted XI.
- After official XI: lineup status = `CONFIRMED`; recompute q and edge using the latest ask price.
- If official XI is still missing at T-55, write `LINEUP_NOT_CONFIRMED`, do not upgrade stake; only reduce or keep.
- Because official XI appears around T-60, there is usually only a 30-40 minute execution window. Cron should fire around T-70 to get ready and then act when the confirmed XI appears.

## Output requirement

For each reforecast, include:

```text
Lineup status: PROVISIONAL / CONFIRMED / LINEUP_NOT_CONFIRMED
Source: FIFA / team official / Sofascore / FotMob / other
Timestamp: ...
Changes from previous forecast: upgrade / keep / downgrade / cancel
New q / ask p / edge / v2 action: ...
```
