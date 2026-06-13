# Order Time UI Implementation Guide

Status: Readdy handoff package
Date: 2026-06-13

Audience:

- Readdy UI sandbox implementer
- Codex reviewer
- LensBank owner reviewing UI scope

Hard boundary:

- UI sandbox only.
- Do not deploy.
- Do not modify DB.
- Do not modify Supabase Edge Functions.
- Do not modify rental fee, payable, Urent, or prepaid/stored-value calculations.
- Do not claim Supabase Branch `ydubnjompnybshscosfd` runtime validation; Codex owns that validation.

## A. Frontend Checkout

Target route:

- `/account/checkout`

### UI To Add

Add two required customer-facing time fields near the existing rental date fields:

- Requested pickup time
- Requested return time

Recommended placement:

- Keep existing pickup date and return date fields.
- Place requested pickup time adjacent to or immediately below pickup date.
- Place requested return time adjacent to or immediately below return date.
- Use a select/time-picker style control that emits zero-padded 24-hour values.

### API To Use

Use the existing checkout order creation path / create-order flow already used by `/account/checkout`.

Add these optional request fields to the order creation payload:

```ts
requestedPickupTime: string; // HH:mm
requestedReturnTime: string; // HH:mm
```

Backend compatibility:

- Old payloads without these fields remain accepted by the backend contract.
- New UI should require these fields for normal customer checkout.

### Field Names

Frontend request body:

- `requestedPickupTime`
- `requestedReturnTime`

Database fields written by backend:

- `orders.requested_pickup_time`
- `orders.requested_return_time`

Time format:

- `HH:mm`
- 24-hour
- zero-padded
- valid example: `09:00`
- invalid examples: `9:00`, `25:99`, `abc`

### Checkout Display Rules

- Treat requested time as customer preferred/reservation time.
- Do not show it as staff-confirmed time.
- If store business-hour data is already safely available, filter time options by pickup/return store hours.
- If store business-hour data is not safely available, provide reasonable options and rely on backend validation/operations review.
- If the selected period may exceed 24 hours, show a soft notice that staff may confirm additional charges later.

### Checkout Forbidden Items

Do not:

- change rental day calculation.
- change rental fee calculation.
- change deposit calculation.
- change `orders.total_price`.
- change `order_items.days`.
- change `order_items.subtotal`.
- change Urent calculation.
- change prepaid/stored-value deduction.
- auto-add late/overtime fee.
- create fee adjustment rows.
- add adjustment UI.
- write directly to `orders` from the browser.
- write directly to `order_items` from the browser.

## B. Backend Order Detail

Target route:

- `/admin/orders/:id`

### Fields To Display

Requested time section:

| Label | Source |
|---|---|
| Requested pickup date | existing `orders.start_date` |
| Requested pickup time | `orders.requested_pickup_time` |
| Requested return date | existing `orders.end_date` |
| Requested return time | `orders.requested_return_time` |

Confirmed time section:

| Label | Source |
|---|---|
| Confirmed pickup time | `orders.confirmed_pickup_time` |
| Confirmed return time | `orders.confirmed_return_time` |
| Time confirmed by | `orders.time_confirmed_by`, if already available in current order model/API |
| Time confirmed at | `orders.time_confirmed_at`, if already available in current order model/API |

Legacy/null behavior:

- Existing orders may have null requested or confirmed time values.
- Null values must render as empty, not set, or pending confirmation.
- Null values must not crash the page.
- Null values must not block existing admin order status workflows.

### APIs To Use

For display of order dates/times:

- Use the existing admin order detail data source if it already fetches these fields.
- If fields are missing from the frontend model, add them to the existing read model/types only.

For payable summary:

- Use `admin-get-order-payable-summary`; see section C.

For confirmed time writes:

- Phase 4A Readdy sandbox should not introduce new write UI unless Codex has already provided and approved a wrapper for `admin-update-order-times`.
- Do not use browser direct `.from('orders').update(...)` for core order time fields.

### Admin Order Detail Forbidden Items

Do not:

- add fee adjustment create/void UI.
- add late fee write UI.
- add overtime fee write UI.
- add damage fee write UI.
- add delivery/manual adjustment write UI.
- mutate `orders.total_price`.
- mutate `order_items.days`.
- mutate `order_items.subtotal`.
- use direct browser table updates for core order/pricing fields.
- infer final payable from local data.

## C. Payable Summary

Target route:

- `/admin/orders/:id`

### Only Data Source

The only allowed frontend data source for final payable display is:

```text
admin-get-order-payable-summary
```

Endpoint:

```text
POST /functions/v1/admin-get-order-payable-summary
```

Request:

```json
{
  "order_id": "<order uuid>"
}
```

`orderId` camelCase may also be accepted by backend, but the preferred UI contract is `order_id`.

### Required Call Pattern

Preferred:

```ts
const { data, error } = await supabase.functions.invoke('admin-get-order-payable-summary', {
  body: { order_id: orderId },
});
```

Fallback if local helper cannot use `functions.invoke`:

```ts
const session = await supabase.auth.getSession();
const accessToken = session.data.session?.access_token;

const response = await fetch(`${supabaseUrl}/functions/v1/admin-get-order-payable-summary`, {
  method: 'POST',
  headers: {
    Authorization: `Bearer ${accessToken}`,
    apikey: supabaseAnonKey,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ order_id: orderId }),
});
```

Never log:

- access token
- Authorization header
- anon key
- service role key
- JWT
- database URL

### Response Schema

Expected response:

```ts
type OrderPayableSummaryResponse = {
  ok: true;
  actor: {
    role: string;
    scope: 'global' | 'assigned_location' | string;
    assigned_location_id: string | null;
  };
  summary: {
    order_id: string;
    order_number: string | null;
    base_rental_total: number;
    base_rental_total_source: string | null;
    order_items_subtotal_total: number;
    deposit_total: number;
    active_fee_adjustments_total: number;
    active_discount_adjustments_total: number;
    active_adjustments_net_total: number;
    voided_fee_adjustments_total: number;
    voided_discount_adjustments_total: number;
    active_adjustment_count: number;
    voided_adjustment_count: number;
    final_payable_total: number;
    calculation_note: string | null;
  };
  adjustments: Array<{
    id: string;
    order_id: string;
    adjustment_type: string;
    amount: number;
    reason: string | null;
    discount_eligible: boolean;
    prepaid_eligible: boolean;
    included_in_urent_share: boolean;
    created_by: string | null;
    created_at: string | null;
    voided_at: string | null;
    is_active: boolean;
  }>;
  guardrails: {
    canonical_read_path: 'get_order_payable_summary';
    frontend_service_role_required: false;
    orders_total_price_unchanged: true;
    order_items_unchanged: true;
    no_urent_change: true;
    no_prepaid_change: true;
    voided_adjustments_excluded_from_final_payable: true;
  };
};
```

Error response:

```ts
type OrderPayableSummaryError = {
  ok: false;
  code: string;
  message: string;
};
```

### Display Rules

Show:

- Base rental total: `summary.base_rental_total`
- Active fee adjustments total: `summary.active_fee_adjustments_total`
- Active discount adjustments total: `summary.active_discount_adjustments_total`
- Final payable total: `summary.final_payable_total`
- Adjustment list from `adjustments`

Label rules:

- `orders.total_price` must be labeled as base rental total only.
- `summary.base_rental_total` must be labeled as base rental total only.
- Final payable total must come from `summary.final_payable_total`.
- Voided adjustments may be visible in history/audit style display only.
- Voided adjustments must not appear as currently payable.

### Payable Summary Forbidden Items

Do not calculate final payable in frontend code.

Forbidden examples:

```ts
const finalPayable = order.total_price + adjustmentsTotal;
```

```ts
const finalPayable = orderItems.reduce(...) + activeAdjustments.reduce(...);
```

```ts
const finalPayable = Number(order.total_price) + Number(lateFee);
```

Do not query browser-direct:

- `order_payable_summary`
- `get_order_payable_summary`
- `order_fee_adjustments` for final payable calculation

Do not:

- use service role key in frontend.
- add adjustment write buttons.
- mutate fee adjustment rows.
- mutate payable totals.

## D. Test Cases And Runtime Ownership

Readdy should use these as UI acceptance cases only. Readdy is not responsible for proving that a preview is connected to Supabase Branch `ydubnjompnybshscosfd`.

Codex Runtime Validation owns:

- Supabase Branch target verification.
- Staging create-order smoke.
- Staging payable summary smoke.
- Cloudflare preview environment verification.
- Any proof that runtime traffic is using `ydubnjompnybshscosfd`.

### Checkout

1. Legacy payload compatibility:
   - Submit checkout using old payload shape without requested time.
   - Expected: backend remains compatible.

2. New requested time payload:
   - Submit checkout with `requestedPickupTime: '09:00'`.
   - Submit checkout with `requestedReturnTime: '18:00'`.
   - Readdy expected: payload shape is correct and UI does not send empty strings.
   - Codex runtime expected: order creation succeeds against Staging and writes `orders.requested_pickup_time = '09:00'`, `orders.requested_return_time = '18:00'`.

3. Invalid time format:
   - Try `9:00`, `25:99`, `abc`.
   - Expected: invalid values are rejected or prevented by UI validation before submit.

4. Pricing guard:
   - Confirm time selection does not change:
     - rental days
     - rental fee
     - deposit
     - `orders.total_price`
     - `order_items.days`
     - `order_items.subtotal`

### Admin Order Detail

1. Legacy/null order:
   - Open an old order with null time fields.
   - Expected: page renders.
   - Expected: null time values show empty/pending state.

2. New requested-time order:
   - Open an order with requested pickup/return time.
   - Expected: requested times display.
   - Expected: confirmed times display if present.

3. No direct update:
   - Inspect code/diff.
   - Expected: no new direct `.from('orders').update(...)` for order time/pricing fields.

### Payable Summary

1. Load payable summary:
   - Open `/admin/orders/:id`.
   - Expected: UI calls `admin-get-order-payable-summary`.
   - Expected: summary loads without frontend calculation.

2. Base-only order:
   - Active adjustment count is zero.
   - Expected: final payable equals base rental total as returned by API.

3. Adjustment display:
   - If adjustments exist, display active and voided state clearly.
   - Expected: voided adjustments do not appear payable.

4. Secret safety:
   - Browser console/network logs must not print JWT, access token, service role key, anon key, or database URL.

## E. Production Forbidden Items

Do not:

- deploy production.
- publish Readdy to production.
- overwrite Cloudflare Pages production.
- change production Supabase DB.
- apply production migration.
- deploy production Edge Functions.
- change DNS/custom-domain routing.
- merge to main without Codex review and owner approval.
- introduce frontend payable calculation.
- introduce frontend rental calculation changes.
- introduce frontend Urent calculation changes.
- introduce frontend prepaid/stored-value changes.
- introduce fee adjustment write UI.

Production movement requires a separate owner-approved gate.
