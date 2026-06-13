# Account2 add-50% execution readback pattern (2026-06-13)

## Trigger

Use when the user says account2 should mirror or increase positions after account1, e.g.:

```text
account 2 增加 50% 仓位
```

## Account2 execution quirks observed

Account2 was executed through a standalone source script that embeds constants rather than the standard account1 `.env` path. Keep secrets out of chat and logs.

Important operational quirk: `create_or_derive_api_key()` may print or surface a non-fatal API-key creation error, and an FOK order attempt may raise an exception while the trade still later appears in `positions`/`activity`. In this session, the first account2 Switzerland-win add order raised an FOK-related exception, but readback showed it had actually filled:

```text
Switzerland win add: size 396 @ 0.82
activity tx: 0x<redacted_64_hex_hash>
positions size moved from 802 -> 1198
```

## Required pattern after any account2 exception

Before retrying a failed-looking account2 order:

1. Query account2 `positions` for the target market.
2. Query account2 recent `activity` and look for the exact market/outcome/side/size/price/tx.
3. If the intended size is already reflected, treat the order as filled and **do not retry** that leg.
4. Only retry the missing legs.
5. In the final report, explicitly say that the exception was reconciled by positions/activity readback.

## Add-50 worked example

Starting account2 relevant positions:

```text
Switzerland win: size 802, initialValue 649.62
Switzerland -1.5: size 604, initialValue 350.32
Brazil win: size 544, initialValue 315.52
Brazil Under 2.5: size 167.5, initialValue 92.125
```

Executed add-50 style:

```text
Switzerland win: +396 @ 0.82, notional 324.72, filled via readback
Switzerland -1.5: +292 @ 0.60, notional 175.20, matched
Brazil win: +345 @ 0.59, notional 203.55, matched
Brazil Under 2.5: no add because it was downgraded/protection-only
```

Final readback:

```text
Switzerland win: size 1198, initialValue 974.3393
Switzerland -1.5: size 896, initialValue 525.5192
Brazil win: size 889, initialValue 515.62
Brazil Under 2.5: size 167.5, initialValue 92.125
```

## Reporting rule

When mirroring account1’s market-selection logic to account2, restate the logic briefly:

- Switzerland: add to existing win + spread structure.
- Brazil: because Under 2.5 was downgraded to protection/observation, add only Brazil win unless the user explicitly requests adding Under.

Do not expose private key, API secret, passphrase, or full credential file contents.
