---
name: Onboard an account owner and collect tax documentation
description: Create a TaxBit account owner and account, then collect and verify W-9 / W-8 tax documentation on their behalf.
api: TaxBit API
docs: https://apidocs.taxbit.com
auth: Bearer token — tenant-scoped for setup, account-owner-scoped for tax-doc submission
operations:
  - post_account-owners
  - post_accounts
  - post_oauth-account-owner-token
  - post_account-owners-id-tax-documentation-data-w-9
  - post_account-owners-id-tax-documentation-data-w-8ben
  - get_account-owners-id-tax-documentation-status
  - get_account-owners-id-us-tin-validation-status
---

# Onboard an account owner and collect tax documentation

Use this flow to bring a user into TaxBit and collect their IRS tax documentation
(W-9 for US persons, W-8BEN/W-8BEN-E for non-US), then confirm it validated.

## Prerequisites
- A **tenant-scoped bearer token** (`post_oauth-token`, valid 24h) for setup calls.
- Send `Authorization: Bearer <token>` on every request.

## Steps

1. **Create the account owner** — `post_account-owners`.
   Rate limit is 50 RPS by default; contact your Implementation Manager for more.
   Keep the returned account owner id (or use your own system identifier).

2. **Create an account** for the owner — `post_accounts`.
   Accounts are what later carry transactions, inventory and documents.

3. **Mint an account-owner-scoped token** — `post_oauth-account-owner-token`.
   This token authorizes submitting/accessing tax documentation *on behalf of*
   that account owner and is required for the W-8/W-9 endpoints. Handle
   expiration per the docs (re-mint on 401).

4. **Submit tax documentation** for the owner:
   - US person → `post_account-owners-id-tax-documentation-data-w-9`
   - Non-US individual → `post_account-owners-id-tax-documentation-data-w-8ben`
   - Non-US entity → `post_account-owners-id-tax-documentation-data-w-8ben-e`
   - Otherwise → `post_account-owners-id-tax-documentation-data-self-certification`

5. **Check status** — `get_account-owners-id-tax-documentation-status`.
   Returns the latest status across all form types (W-9/W-8, Digital Platform
   Seller, Self-Certification). For US persons also poll
   `get_account-owners-id-us-tin-validation-status`.

6. **Resolve validation issues (curing)** if a W-8 needs correction: drive the
   curing component and read OPEN / IN_REVIEW / RESOLVED status from the hook,
   server-side, or webhooks (see asyncapi/taxbit-webhooks.yml).

## Notes
- Prefer the embedded React/browser SDK (components/taxbit-components.yml) to
  collect form data client-side; it uses the same account-owner-scoped token.
- Writes addressed by your system identifiers are idempotent (safe to retry).
