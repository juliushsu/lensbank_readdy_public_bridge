# LensBank Order Time And Fee Adjustment Contract

Status: Phase 3A staging admin API foundation
Date: 2026-06-13

Scope:

- Define the canonical meaning of pickup/return times for orders.
- Define fee adjustment boundaries so late/overtime fees do not leak into rental fee, Urent revenue share, or prepaid deduction logic.
- Provide staging-safe API and admin write contracts before any Readdy UI work.

Non-goals:

- No production DB apply.
- No production deploy.
- No Readdy preview overwrite.
- No rental core rewrite.
- No Urent revenue calculation change.
- No prepaid/stored value core deduction change.

## 1. Current State

`orders.start_date` and `orders.end_date` are date-only fields. They currently drive:

- customer checkout rental period;
- `create-order` rental day calculation;
- `order_items.days` and `order_items.subtotal`;
- admin order detail date display and edit flow;
- Urent income calculation fallback;
- calendar sync defaults.

There is no canonical order-level pickup/return time in Phase 0. Any calendar time such as `10:00` or `18:00` is a synthetic default, not an order source of truth.

## 2. Phase 1 Fields

Phase 1 adds nullable time-only fields to `orders`. These fields are paired with existing date-only columns and do not change pricing by themselves.

| Field | Type | Writer | Meaning |
|---|---:|---|---|
| `requested_pickup_time` | `time` | customer checkout / create-order | Customer's preferred pickup time for `orders.start_date`. |
| `requested_return_time` | `time` | customer checkout / create-order | Customer's preferred return time for `orders.end_date`. |
| `confirmed_pickup_time` | `time` | admin API/RPC | Staff-confirmed official pickup time for `orders.start_date`. |
| `confirmed_return_time` | `time` | admin API/RPC | Staff-confirmed official return time for `orders.end_date`. |
| `time_confirmed_by` | `uuid` | admin API/RPC | Admin profile that last confirmed or adjusted official times. |
| `time_confirmed_at` | `timestamptz` | admin API/RPC | Server timestamp when official times were last confirmed or adjusted. |

Phase 1 display precedence:

1. Show confirmed time when present.
2. Otherwise show requested time when present.
3. Otherwise show date-only fallback and label time as not set.

Phase 1 pricing rule:

- Adding or changing any Phase 1 time field must not automatically change `orders.total_price`, `orders.total_amount`, `order_items.days`, or `order_items.subtotal`.

## 3. Phase 2 Reserved Fields

Phase 2 may add datetime fields for operational reality and late-fee assessment:

| Field | Type | Meaning |
|---|---:|---|
| `actual_pickup_at` | `timestamptz` | Actual handoff timestamp when equipment leaves LensBank custody. |
| `actual_return_at` | `timestamptz` | Actual return timestamp when equipment is received by LensBank staff. |

Timezone policy:

- Canonical business timezone is `Asia/Taipei`.
- Date/time UI should display in `Asia/Taipei`.
- If Phase 2 stores `*_at` as `timestamptz`, API code must construct it from date + time + `Asia/Taipei`, not from browser-local timezone assumptions.
- Existing `start_date` / `end_date` remain the date-level rental period until a later pricing contract explicitly replaces them.

## 4. Fee Classification

| Fee type | Meaning | Included in rental fee | Included in deposit | Included in Urent share | Default prepaid eligible |
|---|---|---:|---:|---:|---:|
| `rental_fee` | Equipment rental fee based on order items and rental days. | yes | no | yes, when item is Urent | yes, through `orders.total_price` |
| `deposit` | Refundable/security deposit. | no | yes | no | no |
| `delivery_fee` / vehicle fee | Vehicle, logistics, or delivery fee. | no | no | no | follows current `orders.total_price` behavior unless moved to adjustment later |
| `late_return_fee` | Late return penalty after staff review. | no | no | no | no |
| `overtime_fee` | Overtime fee after staff review. | no | no | no | no |
| `damage_fee` | Damage compensation. | no | no | no | no |
| `manual_adjustment` | Other staff-entered charge. | no by default | no | no | no |
| `discount_adjustment` | Manual discount/credit adjustment. | no by default | no | no | no |

Fee adjustments belong in `order_fee_adjustments`, not in `order_items`.
`discount_adjustment.amount` is still stored as a non-negative amount; API/display code decides that this type reduces the collected balance.

## 5. Urent Principles

- Urent revenue share is calculated only from rental fee / `order_items`.
- Urent must not read or sum `order_fee_adjustments` by default.
- `late_return_fee` and `overtime_fee` default to `included_in_urent_share=false`.
- `damage_fee`, `delivery_fee`, `manual_adjustment`, and `discount_adjustment` default to `included_in_urent_share=false`.
- Any future exception must be explicit, audited, and covered by separate regression smoke.

## 6. Stored Value / Prepaid Principles

- Existing prepaid/stored value deduction remains based on `orders.total_price`.
- `order_fee_adjustments.prepaid_eligible` defaults to `false`.
- Fee adjustments must not be automatically deducted from prepaid balance.
- If staff later allow a specific adjustment to be paid by stored value, the API must use the row-level `prepaid_eligible` flag and write an audit log.
- Stored value discount logic must not silently discount late return, overtime, or damage fees.

## 7. Frontend Customer Principles

- Customer checkout should require requested pickup and return time after Phase 1 UI is enabled.
- Requested pickup time must be filtered by the selected pickup store's business hours.
- Requested return time must be filtered by the selected return store's business hours.
- If `location_business_hours` is unavailable or incomplete, the UI should show a conservative warning and avoid pretending the time is store-confirmed.
- If selected date + time exceeds 24 hours, checkout should show "may incur additional fees" copy only.
- Checkout must not automatically add late/overtime fees or change rental price because of time selection.

## 8. Admin Principles

- Admin users can confirm or adjust official pickup/return times through a mediated API/RPC.
- Long-term admin writes should not directly update `orders` from the browser for operationally sensitive fields.
- Admin users can add fee adjustments through a mediated API/RPC.
- Admin time changes must not automatically recalculate rental fee unless a later pricing contract explicitly says so.
- Fee adjustments must be separate, itemized, and auditable.
- Voiding a fee adjustment must mark it voided; it must not delete the row.

## 9. Backward Compatibility

- Existing orders may have all time fields as `null`.
- Existing orders continue to display and operate with `start_date` / `end_date`.
- Missing time fields must not block old order status transitions, contract display, prepaid deduction, Urent calculation, or admin detail rendering.
- New create-order requests without requested times remain valid during Phase 1 rollout.
- Phase 1 UI may require time only after the backend and staging smoke are accepted.

## 10. Create Order API Contract Draft

Request additions:

```ts
interface CreateOrderRequest {
  startDate: string;
  endDate: string;
  pickupLocationId: string;
  returnLocationId: string;
  needVehicle?: boolean;
  notes?: string;
  cartItems: CartItemInput[];

  // Phase 1 optional during rollout.
  requestedPickupTime?: string; // HH:mm
  requestedReturnTime?: string; // HH:mm
}
```

Validation:

- Accept absent `requestedPickupTime` / `requestedReturnTime` for backward compatibility.
- When provided, validate `HH:mm` with `00:00` through `23:59`.
- If one requested time is provided by a Phase 1 UI, the UI should provide both.
- API may accept one-sided values during compatibility rollout, but should return a warning.
- If `location_business_hours` has a matching weekday row:
  - pickup time should be within pickup location hours;
  - return time should be within return location hours.
- If business-hour data is missing, closed, malformed, or incomplete, return a non-blocking warning first. Do not reject Phase 1 requests solely because business-hour data is unavailable.

Insert behavior:

```ts
orders.insert({
  start_date: startDate,
  end_date: endDate,
  pickup_location_id: pickupLocationId,
  return_location_id: returnLocationId,
  requested_pickup_time: requestedPickupTime ?? null,
  requested_return_time: requestedReturnTime ?? null,
  // existing price/deposit/status fields unchanged
})
```

Response addition:

```ts
return {
  success: true,
  orderId,
  orderNumber,
  requestedPickupTime: requestedPickupTime ?? null,
  requestedReturnTime: requestedReturnTime ?? null,
  warnings: timeValidationWarnings,
  // existing response fields unchanged
}
```

Non-regression rules:

- Do not change `total_price` calculation.
- Do not change `total_amount` calculation.
- Do not change `order_items.days`.
- Do not change `order_items.subtotal`.
- Do not change prepaid member detection.
- Do not invoke Urent logic during order creation.

## 11. Admin API / RPC Contract Draft

Recommended mediated write surfaces:

| API/RPC | Purpose |
|---|---|
| `update_order_times` | Confirm or adjust `confirmed_pickup_time`, `confirmed_return_time`, `time_confirmed_by`, and `time_confirmed_at`. |
| `add_order_fee_adjustment` | Add a non-voided adjustment row for late return, overtime, damage, delivery, manual charge, or discount adjustment. |
| `void_order_fee_adjustment` | Mark an adjustment voided; never delete adjustment rows. |

Role policy draft:

| Role | `update_order_times` | `add_order_fee_adjustment` | `void_order_fee_adjustment` |
|---|---:|---:|---:|
| `owner` | yes, all orders | yes, all types | yes |
| `super_admin` | yes, all orders | yes, all types | yes |
| `store_manager` | yes, in-scope store orders | yes, operational fee types in-scope | limited; recommended same-store and same-day only unless owner approves broader |
| `staff` / `part_time` | no in Phase 1, or propose-only UI | no in Phase 1 | no |

Store-manager scope:

- In-scope if the order pickup or return location matches the manager's assigned location.
- Cross-store orders should be writable only when the actor is assigned to either involved store, or escalated to owner/super_admin.

Audit requirements:

- `update_order_times` must write an `order_logs` entry with old/new requested and confirmed times, actor id, and safe metadata.
- `add_order_fee_adjustment` must write an `order_logs` entry with adjustment id, type, amount, and eligibility flags.
- `void_order_fee_adjustment` must write an `order_logs` entry with adjustment id and reason.
- Audit metadata must not contain JWTs, auth headers, customer personal documents, payment details, or raw secrets.
- A later Phase 2 may add a dedicated append-only `order_fee_adjustment_events` table if adjustment history needs richer structure.

Phase 3A concrete Edge Function contract:

- See `docs/contracts/ORDER_ADMIN_TIME_AND_ADJUSTMENT_API_CONTRACT.md`.
- Phase 3A chooses service-role Edge Functions with JWT/admin profile checks because this matches the existing project write-path pattern.
- The concrete source endpoints are:
  - `admin-update-order-times`
  - `admin-add-order-fee-adjustment`
  - `admin-void-order-fee-adjustment`
- These endpoints must not be called from Readdy UI until staging runtime smoke passes.

## 12. Regression Smoke Contract

Required smoke before production cutover:

1. Old checkout payload without time still creates an order.
2. New checkout payload with requested time creates an order and writes requested times.
3. Total price, total amount, `order_items.days`, and `order_items.subtotal` match date-only baseline.
4. Urent income calculation does not read `order_fee_adjustments`.
5. Prepaid processing does not automatically deduct adjustments.
6. Admin order detail renders legacy orders with `null` time fields.
7. More-than-24-hour requested duration shows warning only and does not add a fee.
8. Business-hours missing data returns warning, not hard failure.

## 13. Phase 3C Payable Summary Contract

See `docs/contracts/ORDER_PAYABLE_SUMMARY_CONTRACT.md`.

Fee adjustments remain separate from `orders.total_price`.

Final payable display must use `order_payable_summary` or a mediated API backed by it.

`orders.total_price` is base rental total, not final payable total.
