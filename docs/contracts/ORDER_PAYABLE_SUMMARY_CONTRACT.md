# LensBank Order Payable Summary Contract

Status: Phase 3D staging readiness
Date: 2026-06-13

## Purpose

Phase 3B proved that fee adjustments are intentionally separate from `orders.total_price`. Phase 3C defines a canonical payable summary so admin UI, customer UI, and future reports do not mistake base rental total for final payable total.

## Canonical DB Object

Phase 3C uses a read model view plus service-role RPC wrapper:

- `public.order_payable_summary`
- `public.get_order_payable_summary(p_order_id uuid)`

The view is the canonical calculation. The RPC is a stable one-order read surface for mediated backend/API usage.

Phase 3D adds the canonical admin read Edge Function:

- `admin-get-order-payable-summary`

Frontend and browser clients must use the Edge Function, not the service-role RPC directly.

## Calculation Fields

| Field | Meaning |
|---|---|
| `base_rental_total` | Base rental total. Uses `orders.total_price` when present, otherwise falls back to `sum(order_items.subtotal)`. |
| `base_rental_total_source` | Explains whether the base came from `orders.total_price`, `order_items.subtotal`, or zero fallback. |
| `order_items_subtotal_total` | Sum of `order_items.subtotal`, useful for reconciliation. |
| `deposit_total` | Deposit total from `orders.total_deposit`, falling back to item deposit snapshots. |
| `active_fee_adjustments_total` | Sum of active non-discount adjustments where `voided_at is null`. |
| `active_discount_adjustments_total` | Sum of active `discount_adjustment` rows where `voided_at is null`. |
| `active_adjustments_net_total` | `active_fee_adjustments_total - active_discount_adjustments_total`. |
| `voided_fee_adjustments_total` | Sum of voided non-discount adjustments for audit only. |
| `voided_discount_adjustments_total` | Sum of voided discount adjustments for audit only. |
| `active_adjustment_count` | Count of active adjustment rows. |
| `voided_adjustment_count` | Count of voided adjustment rows. |
| `final_payable_total` | `base_rental_total + active_fee_adjustments_total - active_discount_adjustments_total`. |
| `calculation_note` | Human-readable calculation boundary. |

## Rules

- `orders.total_price` remains the base rental total in Phase 3C.
- `orders.total_price` must not be overwritten by fee adjustment changes.
- Active adjustments are rows where `order_fee_adjustments.voided_at is null`.
- Voided adjustments are audit-only and must not be counted in `final_payable_total`.
- `discount_adjustment.amount` is stored as a non-negative amount and reduces the payable summary through `active_discount_adjustments_total`.
- Late return, overtime, damage, delivery, and manual adjustments increase payable total.
- Urent revenue share must continue to use rental fee / order items only, not payable summary.
- Prepaid/stored value deduction must not automatically include fee adjustments until a separate contract allows it.

## API / Frontend Contract

UI must not compute final payable itself.

Admin UI should display:

- `base_rental_total` as "base rental total" or equivalent copy.
- `active_fee_adjustments_total` and `active_discount_adjustments_total` as itemized adjustment totals.
- `final_payable_total` as the final payable value.
- `voided_*` totals only in audit/history sections.

UI must not label `orders.total_price` as final payable total.

Phase 3C does not expose this view directly to browser clients. It is granted to `service_role` only. Phase 3D exposes the mediated admin read path through `admin-get-order-payable-summary`, which verifies admin JWT and order scope before reading the RPC internally.

## Known Phase 3D Gaps

- `order_fee_adjustments` currently has `voided_at` but does not yet have `voided_by` or `void_reason`.
- A dedicated append-only fee adjustment audit table is still recommended.
- Read-only roles below `store_manager` are not enabled until a separate permission contract is approved.
