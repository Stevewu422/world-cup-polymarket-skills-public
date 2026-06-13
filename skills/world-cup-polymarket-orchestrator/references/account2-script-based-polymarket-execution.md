# Account2 script-based execution and verification runbook

## Trigger

Use when the owner provides a Polymarket account2 Python script rather than a normal `.env.accounts/account2.env`, and explicitly asks to use that script for live testing or live execution.

## Core lesson

Some uploaded account2 scripts can place orders by using `py_clob_client_v2` and `create_or_derive_api_key()` even when the API-key creation endpoint logs a non-fatal warning such as:

```text
Could not create api key
```

Do not treat that warning alone as final failure. Continue to inspect whether the later order response returns `success: true` / `status: matched`, then verify via positions/activity.

## Safe execution pattern

1. Save the uploaded script under:
   ```text
   /opt/polymarket-worldcup/.env.accounts/account2_source_<label>.py
   ```
   Set permissions to `600`.
2. Inspect the script without printing secrets:
   - assigned variable names;
   - whether it contains `PRIVATE_KEY`, `FUNDER`, `TOKEN_ID`, `CHAIN_ID`;
   - imports/client version.
3. Confirm HK egress before any live order.
4. Check the script's `TOKEN_ID` orderbook. If no orderbook exists, do not run it as-is.
5. If the owner explicitly authorizes live execution from the script, run it from the HK server and redact long tokens/hashes in chat output.
6. Always follow with verification:
   - query `positions` for the script's `FUNDER` and `TOKEN_ID`;
   - query recent `activity` for the same token;
   - report market title, outcome, size, avg price, initial value/cost, current value, PnL, order id, and tx hash.

## Account2 large-order pattern used in this session

When converting an account2 script to a custom live order rather than running the uploaded script verbatim:

- Load constants from the uploaded script: `HOST`, `PRIVATE_KEY`, `FUNDER`, `CHAIN_ID`.
- Build client:
  ```python
  base = ClobClient(host=HOST, chain_id=CHAIN_ID, key=PRIVATE_KEY,
                    signature_type=1, funder=FUNDER)
  creds = base.create_or_derive_api_key()
  client = ClobClient(host=HOST, chain_id=CHAIN_ID, key=PRIVATE_KEY,
                      creds=creds, signature_type=1, funder=FUNDER)
  ```
- Use fresh Gamma API market data for the target match; do not use the uploaded script's token for unrelated matches.
- Use current orderbook best ask and place GTC buys.
- After execution, wait 10-20 seconds and re-query positions because `initialValue/avgPrice` may initially show zero.

## Reporting nuance

- CLOB response `makingAmount` is the order notional and is usually the clean cost basis.
- `data-api activity.usdcSize` may differ slightly from CLOB notional because of display/fee/indexing semantics.
- Final cost reporting should prefer positions after it has indexed; until then, report CLOB notional and say positions may lag.

## Security note

If credentials were exposed in chat, warn the owner. If the owner still explicitly authorizes use and understands the risk, continue only for owner-directed operations, keep files permissioned `600`, redact secrets in all outputs, and avoid persisting raw secrets outside the necessary server-side file.

Never print private keys, API secrets, or passphrases in final answers or logs delivered to chat.
