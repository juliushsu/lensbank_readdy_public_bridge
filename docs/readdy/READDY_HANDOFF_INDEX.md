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
- time selection does not change rental fee, rental days, deposit, Urent, prepaid, or stored-value behavior.
- `/admin/orders/:id` displays requested and confirmed times without crashing on null values.
- `/admin/orders/:id` includes read-only payable summary.
- payable summary uses `admin-get-order-payable-summary`.
- frontend does not compute final payable.
- no adjustment write UI is introduced.
- no production publish or deploy is performed.

Completion package from Readdy must include:

- modified file list;
- build result;
- ZIP export.
