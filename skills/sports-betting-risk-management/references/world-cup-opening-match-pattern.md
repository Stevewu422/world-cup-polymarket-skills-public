# Example: World Cup first-match betting plan pattern

This reference captures a reusable pattern from a session where the user asked for a World Cup betting strategy starting from the first match.

## Pattern used

1. Verify tournament schedule via a current source/API.
2. Convert kickoff time into Beijing and/or Bangkok time.
3. For the opener, avoid heavy staking because opening matches are high-variance and emotionally priced.
4. Prefer conditional markets over absolute predictions when odds are unavailable.
5. Use bankroll units and stop-loss rules before match picks.

## Example output structure

```text
首场：Mexico vs South Africa
倾向：Mexico not lose, but do not chase low-price outright win.
Preferred markets:
- Mexico Draw No Bet / Asian handicap 0: 1 unit if price is acceptable.
- Mexico -0.25: small stake only if price is good.
- Under 2.75 or Under 3.0: 0.5 unit if lineup and price support it.
Avoid:
- Heavy correct score.
- Large favorite handicap.
- Chasing after first loss.
```

## Reusable rationale

- Opening matches: public attention and host narratives distort odds.
- Group stage round 1: teams often avoid losing more than chasing a big win.
- Without live odds: provide thresholds and no-bet conditions.
