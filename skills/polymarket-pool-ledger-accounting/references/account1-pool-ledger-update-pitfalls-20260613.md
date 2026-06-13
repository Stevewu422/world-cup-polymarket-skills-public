# Polymarket account1 pool ledger update pitfalls (2026-06-13)

Use when updating `/root/.hermes/private_tasks/coco/polymarket_betting_participants/latest_account1_pool.md` after new account1 orders.

## Core rule

Update the pool ledger only for account1 orders. account2 orders remain separate unless the owner explicitly says account2 joins the pool.

## Safe update sequence

1. Backup the current `latest_account1_pool.md` before editing.
2. Parse participant balances from the `余额与占比` section.
3. Compute each participant ratio from balance / total pool balance, not from rounded displayed percentages.
4. For each new account1 market cost, allocate:
   ```text
   participant_market_allocation = market_cost * participant_ratio
   ```
5. Insert new market allocation lines under every participant before that participant’s `合计分摊下注` line.
6. Update each participant total by adding the newly allocated amounts.
7. Update global `account1 下注总成本` and `市场成本合计`.
8. Add an execution record section with market, cost, size, price, orderID, tx, HK execution, order type, and matched status.
9. Verify:
   - participant balances still sum to pool balance;
   - participant allocation totals sum approximately to account1 total cost;
   - account2 orders are not included.

## Editing pitfall

Avoid a broad regex over multiple participant sections. It can accidentally append all new allocation lines into the last participant or corrupt totals.

Safer approach:

- Process the ledger line-by-line.
- Track the current participant after a line starting with `### <name>（占比 ...）`.
- When hitting that participant’s `- 合计分摊下注：... USD`, insert the new per-market allocation lines immediately before it, then update only that one total.

If an edit looks wrong, restore the pre-edit backup, then re-run the line-based update.

## Reporting

After updating the ledger, tell the owner only the relevant facts:

- actual executed notional;
- whether positions/activity confirmed the fills;
- new account1 ledger path;
- if account2 was involved, state explicitly it was saved in a separate account2 record and not merged into the pool.
