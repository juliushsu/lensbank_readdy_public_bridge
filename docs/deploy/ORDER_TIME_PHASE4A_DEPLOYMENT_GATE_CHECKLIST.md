# Order Time Phase 4A Deployment Gate Checklist

Status: checklist only
Date: 2026-06-13

Use this checklist before any Order Time UI moves beyond sandbox or preview. Phase 4A itself does not approve production deployment.

## Environment Gate

- [ ] Target environment is Staging or Cloudflare preview only.
- [ ] Production DB remains untouched.
- [ ] Production Edge Functions remain untouched unless a separate production deploy gate is approved.
- [ ] Readdy preview is used as UI sandbox only.
- [ ] Cloudflare Pages production/custom domain is not overwritten.
- [ ] GitHub canonical branch/commit is recorded.
- [ ] Preview URL is recorded.

## Staging Backend Gate

- [ ] Staging DB schema verified:
  - [ ] `orders.requested_pickup_time`
  - [ ] `orders.requested_return_time`
  - [ ] `orders.confirmed_pickup_time`
  - [ ] `orders.confirmed_return_time`
  - [ ] `orders.time_confirmed_by`
  - [ ] `orders.time_confirmed_at`
  - [ ] `order_fee_adjustments`
  - [ ] `order_payable_summary`
  - [ ] `get_order_payable_summary(order_id)`
- [ ] Staging functions deployed:
  - [ ] `create-order` optional requested time support
  - [ ] `admin-update-order-times`
  - [ ] `admin-add-order-fee-adjustment`
  - [ ] `admin-void-order-fee-adjustment`
  - [ ] `admin-get-order-payable-summary`
- [ ] Staging runtime smoke for payable read API passed.

## Frontend Build Gate

- [ ] Readdy export reviewed by Codex before merge.
- [ ] Accepted UI changes are ported into GitHub canonical source.
- [ ] No Readdy direct production publish.
- [ ] Frontend build passes.
- [ ] Type check/lint pass if available.
- [ ] No service role key is present in frontend code.
- [ ] No JWT, anon key, or Authorization header is logged.
- [ ] Environment variables point to the intended preview/Staging target.

## Checkout Regression Gate

- [ ] Existing checkout flow still renders pickup date and return date.
- [ ] Checkout UI includes requested pickup time.
- [ ] Checkout UI includes requested return time.
- [ ] New time payload uses:
  - [ ] `requestedPickupTime`
  - [ ] `requestedReturnTime`
- [ ] Time format is `HH:mm`.
- [ ] Old checkout payload without requested time remains backend-compatible.
- [ ] New checkout payload with requested time succeeds on Staging.
- [ ] Checkout time selection does not change:
  - [ ] `orders.start_date`
  - [ ] `orders.end_date`
  - [ ] `orders.total_price`
  - [ ] `order_items.days`
  - [ ] `order_items.subtotal`
  - [ ] Urent logic
  - [ ] prepaid/stored-value logic
- [ ] Over-24-hour notice is informational only and does not auto-add charges.

## Admin Order Detail Gate

- [ ] `/admin/orders/:id` renders existing orders with null time fields.
- [ ] Requested pickup time displays from `requested_pickup_time`.
- [ ] Requested return time displays from `requested_return_time`.
- [ ] Confirmed pickup time displays from `confirmed_pickup_time`.
- [ ] Confirmed return time displays from `confirmed_return_time`.
- [ ] Existing order date/status/payment sections still render.
- [ ] Admin order detail smoke passes with a legacy/null-time order.
- [ ] Admin order detail smoke passes with a new requested-time order.
- [ ] No direct browser `.from('orders').update(...)` is introduced for core time/pricing fields.

## Payable Summary Gate

- [ ] Admin order detail includes a read-only payable summary block.
- [ ] Payable summary calls `admin-get-order-payable-summary`.
- [ ] Payable summary does not query `order_payable_summary` directly from browser.
- [ ] Payable summary does not compute final payable in frontend code.
- [ ] `orders.total_price` is labeled as base rental total only.
- [ ] Final payable displays from `summary.final_payable_total`.
- [ ] Active fee adjustments display from `summary.active_fee_adjustments_total`.
- [ ] Active discount adjustments display from `summary.active_discount_adjustments_total`.
- [ ] Adjustment list displays active/voided state clearly.
- [ ] Voided adjustments do not look payable.
- [ ] Payable summary display smoke passes.

## Forbidden UI Gate

Confirm the UI does not include:

- [ ] Add adjustment button.
- [ ] Void adjustment button.
- [ ] Late return fee write UI.
- [ ] Overtime fee write UI.
- [ ] Damage fee write UI.
- [ ] Delivery fee write UI.
- [ ] Manual adjustment write UI.
- [ ] Frontend Urent recalculation.
- [ ] Frontend prepaid/stored-value deduction change.
- [ ] Frontend rental fee/day recalculation from time.
- [ ] Direct mutation of `orders.total_price`.
- [ ] Direct mutation of `order_items.days` or `order_items.subtotal`.

## Production Gate

Production remains blocked until all are true:

- [ ] Owner explicitly approves production DB migration/apply if needed.
- [ ] Owner explicitly approves production Edge Function deploy if needed.
- [ ] Owner explicitly approves frontend production deploy.
- [ ] Cloudflare preview is accepted.
- [ ] Rollback commit and rollback steps are documented.
- [ ] Production smoke plan is documented.

Phase 4A expected production status:

```text
production DB touched: no
production deploy: no
Readdy production publish: no
Cloudflare production overwrite: no
```
