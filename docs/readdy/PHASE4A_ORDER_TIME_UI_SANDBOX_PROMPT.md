# Phase 4A Order Time UI Sandbox Prompt

Status: Readdy UI sandbox prompt
Date: 2026-06-13

## Copy/Paste Prompt For Readdy

You are updating LensBank UI in a Readdy sandbox only. Do not publish to production. Do not modify backend, database, Supabase Edge Functions, RLS, Urent logic, prepaid/stored-value logic, or rental fee calculation.

Target screens:

- `/account/checkout`
- `/admin/orders/:id`

Backend contract is already prepared in Staging:

- `orders.requested_pickup_time`
- `orders.requested_return_time`
- `orders.confirmed_pickup_time`
- `orders.confirmed_return_time`
- `order_fee_adjustments`
- `order_payable_summary`
- Edge Function: `admin-get-order-payable-summary`

Use the existing UI language and design system. Keep the app operational and compact. Do not create a marketing page.

## 1. Checkout UI

On `/account/checkout`, add required customer requested time fields:

- Requested pickup time
- Requested return time

Behavior:

- Keep existing pickup date and return date behavior.
- Add time selectors next to or below the existing date fields.
- Time values must be submitted as `requestedPickupTime` and `requestedReturnTime`.
- Format must be `HH:mm`, zero-padded, 24-hour time.
- The fields are customer requested/preferred times only.
- Do not create confirmed time fields on customer checkout.
- Do not recalculate rental days from time.
- Do not change rental fee, deposit, total price, Urent, prepaid, stored value, or order item subtotal from time selection.
- If selected date/time may exceed 24 hours, show a soft notice that additional fees may be confirmed by staff later. Do not auto-add charges.
- If store business-hour data is available in existing frontend state/API, filter the time options. If not available, show reasonable time options and do not block order creation beyond required HH:mm format.
- Old checkout payload without time must remain compatible where backend accepts it, but new UI should require the fields for normal customer submission.

Do not add:

- fee adjustment UI
- late fee UI
- automatic price changes
- Urent changes
- prepaid/stored-value changes
- direct table updates

## 2. Admin Order Detail UI

On `/admin/orders/:id`, display order time information clearly:

Requested time section:

- Requested pickup date: existing `start_date`
- Requested pickup time: `requested_pickup_time`
- Requested return date: existing `end_date`
- Requested return time: `requested_return_time`

Confirmed time section:

- Confirmed pickup time: `confirmed_pickup_time`
- Confirmed return time: `confirmed_return_time`
- Confirmed by / confirmed at if available from existing data or future API response

Legacy behavior:

- Old orders may have null time values.
- Null requested/confirmed time must render as an empty/unknown state, not crash.
- Do not block existing order status workflows when time is null.

Phase 4A UI is read/display-oriented for admin order detail unless an existing safe Codex-provided API wrapper is already wired. Do not use direct `.from('orders').update(...)` for core order time fields.

## 3. Payable Summary Read-Only Block

Add a read-only payable summary block on `/admin/orders/:id`.

This block must call:

```ts
supabase.functions.invoke('admin-get-order-payable-summary', {
  body: { order_id: orderId }
});
```

If the local wrapper cannot use `functions.invoke`, use `fetch` with the current Supabase session access token and anon key:

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

Never log or render `accessToken`, `Authorization`, anon key, service role key, JWT, or database URLs.

Display from API response only:

- Base rental total: `summary.base_rental_total`
- Active fee adjustments total: `summary.active_fee_adjustments_total`
- Active discount adjustments total: `summary.active_discount_adjustments_total`
- Final payable total: `summary.final_payable_total`
- Adjustment list: `adjustments`

Important labels:

- `orders.total_price` or `summary.base_rental_total` must be labeled as base rental total, not final total.
- Final payable must come from `summary.final_payable_total`.
- Voided adjustments may appear only in an audit/history style list and must not look payable.

Do not calculate:

```ts
finalPayable = totalPrice + adjustments
```

Do not calculate final payable anywhere in frontend code. The backend function is the source of truth.

## 4. Forbidden Changes

Do not:

- deploy production
- publish Readdy to production
- edit Supabase schema
- edit Supabase Edge Functions
- edit RLS policies
- change `orders.total_price`
- change `order_items.days`
- change `order_items.subtotal`
- change Urent income/revenue-share logic
- change prepaid/stored-value deduction logic
- create adjustment add/void buttons
- create late fee, overtime fee, damage fee, delivery fee, or manual adjustment write UI
- write directly to `order_fee_adjustments`
- write directly to core order pricing fields
- invent final payable calculations in the frontend

## 5. Expected Sandbox Output

Return a Readdy sandbox preview/export only.

The exported UI should be handed back to Codex for:

1. ZIP/source diff review.
2. Merge into GitHub canonical branch.
3. Frontend build.
4. Staging preview deploy.
5. Smoke verification.

No production publish is allowed from Readdy.
