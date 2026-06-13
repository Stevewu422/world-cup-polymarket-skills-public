# Pool ledger, market-level allocation, and public betting page workflow

## Trigger

Use when the owner asks to record multiple participants in a Polymarket/World Cup betting pool, split current account bets by person, or prepare a public-facing page showing only recommended betting directions.

## Pool accounting rules from the session

- Keep **account-level execution** separate from **pool accounting**.
- `account1` can be the pool's canonical execution account; `account2` should be excluded from the overall pool statistics unless the owner explicitly says to include it.
- Pool participant balances can change by transfers between people. Example: Steve transfers 443 USDT to 笑笑, so Steve balance decreases by 443 and 笑笑 is added with 443.
- Percentages are based on current participant balance / current pool total.
- For current open bets, split every independent market by each person's pool percentage.
- Do not merge contract share/size across markets. Only money exposure can be summed.

## Market-level split template

For each person:

```text
Person: NAME
Balance: X USD
Pool share: P%

Match / Market A: market_cost * P = allocated USD
Match / Market B: market_cost * P = allocated USD
...
Total allocated bet = sum allocated USD
```

For each match, list independent markets separately. Example for Haiti vs Scotland:

```text
Haiti vs Scotland / Scotland 胜 Yes: wins if Scotland wins the match.
Haiti vs Scotland / Scotland -1.5: wins only if Scotland wins by 2+ goals.
Haiti vs Scotland / Over 2.5: wins if total goals >= 3.
```

## Settlement after a match finishes

When a match ends:

1. Query account-level realized outcome and settlement/PnL by independent market.
2. Compute each market's profit/loss:
   ```text
   market_pnl = payout_or_close_value - market_cost
   ```
3. Allocate market PnL to participants by the pool share that applied to that bet, unless the owner specified a special profit split.
4. Update each person:
   ```text
   new_balance = old_balance + allocated_pnl
   ```
5. Produce:
   - match-level PnL by market;
   - per-person PnL by market;
   - per-person new balance and new pool share;
   - an audit CSV/Markdown file.

## Public web page rule

When the owner wants a page to share with outsiders, create a **public version** only:

- Show every match and recommended betting direction.
- Use scalable units such as "每 100 单位" instead of internal bet amounts.
- Do **not** show account names, account numbers, account1/account2, current positions, friends' balances, pool totals, or internal allocation details.
- Avoid meta text such as "简单版：只展示..." inside the page; make the page itself clean.
- Add a general risk note that the page is pre-match reference only and not guaranteed profit.

Recommended public-page structure:

```text
Title: 世界杯赛前下注建议
For each match:
  - Main direction and units
  - Secondary direction and units
  - Reason in one sentence
  - Avoid list
How to scale: multiply units by intended budget / 100
Risk note
```

## File locations used in this workflow

- Private ledger/audit outputs:
  `/root/.hermes/private_tasks/coco/polymarket_betting_participants/`
- Public shareable HTML copies:
  `/root/.hermes/media_cache/polymarket_public_betting_page/`

## Verification checklist

Before sharing a public page:

1. Search the HTML for internal names: participant names, `account1`, `account2`, `资金池`, `持仓`, balances, and private amounts.
2. Verify the HTML opens locally.
3. If generating an external URL, fetch the URL and confirm the title text appears.
4. Provide the external link and a local `MEDIA:` fallback if appropriate.
