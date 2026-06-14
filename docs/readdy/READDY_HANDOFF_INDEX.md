# Readdy Handoff Index: Order Time Phase 4A

Status: public handoff index
Date: 2026-06-13

## A. Document List

Start here:

- [START_HERE_READDY_ORDER_TIME.md](./START_HERE_READDY_ORDER_TIME.md)

Readdy must read:

- [README_FIRST_FOR_READDY.md](./README_FIRST_FOR_READDY.md)
- [ORDER_TIME_FILE_INDEX.md](./ORDER_TIME_FILE_INDEX.md)
- [ORDER_TIME_UI_IMPLEMENTATION_GUIDE.md](./ORDER_TIME_UI_IMPLEMENTATION_GUIDE.md)
- [PHASE4A_ORDER_TIME_UI_SANDBOX_PROMPT.md](./PHASE4A_ORDER_TIME_UI_SANDBOX_PROMPT.md)
- [VER1147_FAIL_REVIEW.md](./VER1147_FAIL_REVIEW.md)
- [VER1148_REQUIRED_FIX.md](./VER1148_REQUIRED_FIX.md)
- [ORDER_TIME_AND_FEE_ADJUSTMENT_CONTRACT.md](../contracts/ORDER_TIME_AND_FEE_ADJUSTMENT_CONTRACT.md)
- [ORDER_PAYABLE_SUMMARY_CONTRACT.md](../contracts/ORDER_PAYABLE_SUMMARY_CONTRACT.md)
- [ORDER_PAYABLE_READ_API_CONTRACT.md](../contracts/ORDER_PAYABLE_READ_API_CONTRACT.md)

Release planning references:

- [ORDER_TIME_PHASE4A_DEPLOYMENT_GATE_CHECKLIST.md](../deploy/ORDER_TIME_PHASE4A_DEPLOYMENT_GATE_CHECKLIST.md)
- [ORDER_TIME_PHASE4A_CLOUDFLARE_SYNC_PLAN.md](../deploy/ORDER_TIME_PHASE4A_CLOUDFLARE_SYNC_PLAN.md)

## B. Functional Scope

Allowed UI sandbox work:

- Add requested pickup time and requested return time fields to `/account/checkout`.
- Submit `requestedPickupTime` and `requestedReturnTime` in `HH:mm` format.
- Show requested pickup/return time in `/admin/orders/:id`.
- Show confirmed pickup/return time in `/admin/orders/:id`.
- Add read-only payable summary display in `/admin/orders/:id`.
- Call `admin-get-order-payable-summary` for final payable display.

Runtime target boundary:

- Readdy is not required to switch to or verify Supabase Branch `ydubnjompnybshscosfd`.
- Current Readdy connection appears limited to the main Supabase project (`reczunexoejndosqzjal`).
- Codex Runtime Validation owns Staging branch verification after Readdy exports the UI.

## C. Prohibited Scope

Readdy must not:

- modify rental fee calculation.
- modify rental day calculation.
- modify payable calculation.
- calculate final payable in frontend code.
- modify Urent revenue share.
- modify prepaid/stored-value behavior.
- modify DB schema.
- modify migration SQL.
- modify Supabase Edge Functions.
- create adjustment add/void UI.
- use service role key in frontend.
- publish production.
- deploy Cloudflare.
- deploy Supabase.
- merge main.

## D. Acceptance Criteria

The Readdy UI sandbox is acceptable only if:

- `/account/checkout` still supports the existing date flow.
- `/account/checkout` includes requested pickup/return time UI.
- new checkout payload includes `requestedPickupTime` and `requestedReturnTime`.
- time format is `HH:mm`.
- selected requested pickup/return times are cleared if they no longer exist in the latest slots after date/store changes.
- submit is blocked if selected requested pickup/return times do not exist in the latest slots.
- Ver1146 build fix is retained; do not regress to nonexistent `eslint-config-airbnb-typescript` versions or broken AutoImport config.
- time selection does not change rental fee, rental days, deposit, Urent, prepaid, or stored-value behavior.
- `/admin/orders/:id` displays requested and confirmed times without crashing on null values.
- `/admin/orders/:id` includes read-only payable summary.
- payable summary uses `admin-get-order-payable-summary`.
- frontend does not compute final payable.
- no adjustment write UI is introduced.
- no production publish or deploy is performed.
- Readdy does not claim Supabase Branch `ydubnjompnybshscosfd` validation.

Completion package from Readdy must include:

- modified file list;
- build result;
- ZIP export.
