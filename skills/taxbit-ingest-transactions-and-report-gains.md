---
name: Ingest transactions and report realized gains
description: Send an account's transactions to TaxBit, set its accounting method, then read inventory and realized gains (Form 8949 / 1099-B).
api: TaxBit API
docs: https://apidocs.taxbit.com
auth: Bearer token — tenant-scoped
operations:
  - post_transactions-external-id
  - get_accounts-id-transactions
  - post_accounts-id-disposition-methods-history
  - get_inventory
  - get_gains
  - get_gains-summary
  - get_gains-breakdown
  - post_reports-inventory-summary
  - get_reports-inventory-summary-reportId
---

# Ingest transactions and report realized gains

Load an account's activity into TaxBit and read back cost basis, inventory and
realized gains for tax reporting.

## Prerequisites
- A **tenant-scoped bearer token** (`post_oauth-token`).
- An existing account (see the onboarding skill).

## Steps

1. **Set the disposition (accounting) method** for the account —
   `post_accounts-id-disposition-methods-history`. Supported methods include
   FIFO, LIFO, HIFO and LOFO; this determines lot-selection order for gains.

2. **Send transactions** — `post_transactions-external-id`.
   This is create-or-update keyed by *your* transaction identifier, so retries
   are idempotent (no duplicates). Batch/loop as needed within 50 RPS.
   For transfer-in/out lots with user-entered cost basis, attach
   `post_transfer-lots-transactions-transaction-id`.

3. **Verify ingestion** — `get_accounts-id-transactions` (and
   `get_transactions-external-id-id` for a single transaction).

4. **Read inventory** — `get_inventory` (undisposed lots for one asset; provide
   `asset_id` or `asset_code`) and `get_inventory-summaries` (all assets).

5. **Read realized gains**:
   - `get_gains` — detailed cost basis / proceeds / gain-loss line items
     (correspond to IRS Form 8949 / 1099-B).
   - `get_gains-breakdown` — short-term vs long-term vs total.
   - `get_gains-summary` — total gains for a period.

6. **Generate an inventory-summary report (async)** —
   `post_reports-inventory-summary` returns a `reportId`; then poll
   `get_reports-inventory-summary-reportId` until complete and download.
   Respect the documented inventory SLAs for async processing.

## Notes
- Gains/inventory are per account; income can be summarized via
  `get_accounts-id-income`.
- Reports and inventory are asynchronous — poll, don't block.
