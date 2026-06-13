# Account2 direct-script execution and risk override notes

## Trigger

Use when the user provides a standalone Polymarket Python script for a secondary account and explicitly asks to execute it or use it as the account2 control path.

## Durable lessons from the session

1. The user may deliberately accept risk for a key that appeared in chat and may explicitly instruct execution anyway. Record the risk clearly, but once the user gives explicit override, execute the requested low-scope action and verify.
2. Do not assume `account2.env` is the only usable path. A script using `py_clob_client_v2`, `create_or_derive_api_key()`, `create_and_post_order(...)`, `OrderArgs`, `OrderType`, and `Side` can place orders even when manually copied `POLYMARKET_API_KEY/SECRET/PASSPHRASE` in `account2.env` fail with `401 Unauthorized/Invalid api key`.
3. If `create_or_derive_api_key()` prints `Could not create api key` but the returned/derived creds still allow `post_order`, treat it as a warning and verify with order response + positions/activity, not as automatic failure.
4. Always save uploaded scripts under `.env.accounts/` with `chmod 600` and run from the HK server workdir `/opt/polymarket-worldcup`.
5. For user-visible summaries, never echo private keys or full API credentials. Redact long tokens and hashes in logs shown to chat unless order/tx hash is specifically safe to share.

## Safe execution sequence

```text
1. Copy uploaded script to:
   /opt/polymarket-worldcup/.env.accounts/account2_source_<name>.py
2. chmod 600 the script.
3. Inspect AST for HOST, PRIVATE_KEY, FUNDER, TOKEN_ID, CHAIN_ID, post_order/create_and_post_order.
4. Verify HK egress.
5. Verify target token orderbook if possible.
6. If user explicitly overrides leaked-key risk, run the script or a minimal wrapper using the same credentials.
7. Verify with:
   - order response status=matched/open/failed
   - data-api positions for the funder
   - data-api activity for recent fills
8. Report market, outcome, price, size, notional, orderID/tx, and current position/PnL.
```

## Pitfalls

- `account2.env` auth failure does not prove the direct script cannot trade; the direct script may derive API credentials on the fly.
- The script's `TOKEN_ID` may point to a different market than the current task. For task-specific orders (e.g. USA vs Paraguay 1000 USDC), use the script's account credentials but fetch fresh Gamma tokens for the requested market.
- Do not mix account1 `.env` into account2 runs. Explicitly parse the account2 source script or account2 env path.
