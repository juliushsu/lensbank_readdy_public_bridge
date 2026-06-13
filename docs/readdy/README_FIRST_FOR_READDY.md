# Read Me First For Readdy

Status: required first-read
Date: 2026-06-13

## You Are In UI Sandbox Mode Only

Readdy may create a UI sandbox for Order Time Phase 4A.

Readdy must not publish production.

Readdy output must be exported and reviewed by Codex before any merge, build, preview, or release.

## What Readdy Can Do

Readdy can propose UI-only changes for:

- `/account/checkout`
- `/admin/orders/:id`

Allowed UI scope:

- Add customer requested pickup time field.
- Add customer requested return time field.
- Display requested pickup/return time on admin order detail.
- Display confirmed pickup/return time on admin order detail.
- Display read-only payable summary on admin order detail.
- Display adjustment list returned by the payable summary API.
- Improve layout, labels, empty states, loading states, and error states within this scope.

## What Readdy Must Not Do

Do not:

- modify rental fee calculation.
- modify rental day calculation.
- modify payable calculation.
- calculate final payable in frontend code.
- modify Urent revenue-share logic.
- modify prepaid/stored-value logic.
- modify deposit logic.
- modify `orders.total_price`.
- modify `order_items.days`.
- modify `order_items.subtotal`.
- create fee adjustment write UI.
- create late fee write UI.
- create overtime fee write UI.
- create damage fee write UI.
- create delivery/manual adjustment write UI.
- write directly to `orders` core pricing/time fields.
- write directly to `order_fee_adjustments`.
- change Supabase schema.
- change Supabase Edge Functions.
- change RLS.
- add service-role key to frontend.
- log JWTs, access tokens, anon keys, service-role keys, or database URLs.
- publish production.
- overwrite Cloudflare Pages production.
- change DNS/custom-domain settings.

## Required Payable Rule

The only allowed source for final payable display is:

```text
admin-get-order-payable-summary
```

The UI must show:

- `summary.base_rental_total` as base rental total.
- `summary.active_fee_adjustments_total` as active fee adjustments total.
- `summary.active_discount_adjustments_total` as active discount adjustments total.
- `summary.final_payable_total` as final payable total.

The UI must not compute:

```text
final payable = total price + adjustments
```

or any similar frontend formula.

## Required Checkout Rule

Checkout should submit:

```ts
requestedPickupTime: 'HH:mm'
requestedReturnTime: 'HH:mm'
```

These are customer requested times only. They do not change price automatically.

## Required Handoff Rule

After UI sandbox work:

1. Export the Readdy source/ZIP.
2. Do not publish.
3. Give the export to Codex.
4. Codex reviews and ports approved changes into GitHub canonical source.
5. Cloudflare Pages preview comes from GitHub canonical source, not Readdy.

## Read Next

Read these in order:

1. [ORDER_TIME_FILE_INDEX.md](./ORDER_TIME_FILE_INDEX.md)
2. [ORDER_TIME_UI_IMPLEMENTATION_GUIDE.md](./ORDER_TIME_UI_IMPLEMENTATION_GUIDE.md)
3. [PHASE4A_ORDER_TIME_UI_SANDBOX_PROMPT.md](./PHASE4A_ORDER_TIME_UI_SANDBOX_PROMPT.md)
