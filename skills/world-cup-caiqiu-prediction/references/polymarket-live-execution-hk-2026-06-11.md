# Polymarket live execution notes — HK server workflow

## When this reference applies

Use when the user explicitly asks to place live Polymarket football/World Cup orders, especially after giving an amount per pick (e.g. “10 USD each”) or asking to buy match winner / totals markets.

## Durable lessons from 2026-06-11 session

1. Live order execution must run from the Hong Kong server, not the default Hermes host.
   - Server: `<YOUR_HK_SERVER_IP>`
   - Project: `/opt/polymarket-worldcup`
   - Reason: user corrected that default/Singapore egress cannot place the order; live trading should require HK egress verification.

2. Keep public market discovery and live execution separate.
   - Public discovery can use Gamma API.
   - Live order posting must run on HK server with the Polymarket env file and region guard.

3. For World Cup games page markets, Gamma keyset endpoint with sports params exposes child/prop markets that plain search may miss:

```bash
curl -sS -A 'Mozilla/5.0' \
  'https://gamma-api.polymarket.com/events/keyset?tag_id=102232&series_id=11433&closed=false&order=startTime&ascending=true&limit=100&include_best_lines=true'
```

World Cup league constants observed from Polymarket web bundle:
- `tag_id=102232`
- `series_id=11433`
- sports slug: `world-cup`
- gamma league short slug can appear as `fifwc`

4. `clobTokenIds` order maps to `outcomes` order.
   - For binary markets: `outcomes[0]` = YES token, `outcomes[1]` = NO token.
   - For totals O/U markets, Polymarket’s displayed Yes side corresponds to Over for the line; No side corresponds to Under.
   - Example: `O/U 2.5` → buy Under 2.5 by buying the No token (`clobTokenIds[1]`).

5. Always validate the token before live order:

```bash
cd /opt/polymarket-worldcup
source .venv/bin/activate
./pm_worldcup.py orderbook <CLOB_TOKEN_ID>
```

Do not place if orderbook returns `invalid token id`, `No orderbook exists`, no ask, or closed market.

6. For ~10 USDC per pick:
   - Use current best ask as limit price, rounded to market tick size.
   - Size = floor((target_usd / price) * 10000) / 10000.
   - Check `min_order_size` from orderbook; Polymarket often requires at least 5 shares.
   - Then post with explicit live flags only after user has authorized live trading.

7. Required live guardrails:
   - Verify execution region shows `country=HK` and IP `<YOUR_HK_SERVER_IP>` or other user-approved HK egress.
   - Check balance and allowance before the batch.
   - Store logs under `/opt/polymarket-worldcup/logs/`.
   - Report orderID, transaction hash, status, price, size, and balance change.
   - Never echo `.env`, private key, API secret, or passphrase in chat.

8. If the user provides their own Polymarket order script, do not assume it is executable as-is.
   - Upload it to the HK project with restrictive permissions and run syntax/import checks first.
   - Old `py_clob_client` scripts may fail with `invalid order version`; keep the script as a reference but execute live orders through the maintained HK helper (`pm_worldcup.py`) unless the uploaded script passes a live-compatible dry run.
   - Treat hard-coded token IDs in user scripts as stale until orderbook validates them; if orderbook says `invalid token id`, refetch current `clobTokenIds` from Gamma keyset.

## Example live command shape

```bash
cd /opt/polymarket-worldcup
source .venv/bin/activate
./pm_worldcup.py place \
  --token-id '<validated_clob_token_id>' \
  --side BUY \
  --price '<best_ask_or_user_limit>' \
  --size '<computed_size>' \
  --live \
  --yes-i-understand-real-money
```

## Session example pattern

- Match winner: buy selected team’s Yes token from the moneyline market.
- Totals: for `Under 2.5`, buy the No token of the `totals` market with `line=2.5`.
- First-three-games batch: after winner picks are placed, if user says “大小球也要下单”, do not ask again if amount was already specified; locate each game’s `More Markets` totals `O/U 2.5`, validate orderbook, and place the corresponding Under/Over order as instructed.
