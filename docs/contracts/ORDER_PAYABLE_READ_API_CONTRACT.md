# Order Payable Read API Contract

Status: Phase 3D staging foundation
Date: 2026-06-13

## Purpose

`admin-get-order-payable-summary` is the canonical admin read API for order payable totals. Admin UI, Readdy UI sandbox, LINE Bot, AI Agent, Accounting, and future operational tools must not recompute final payable totals independently.

The API wraps the DB canonical calculation:

- `public.get_order_payable_summary(p_order_id uuid)`

## Endpoint

```text
POST /functions/v1/admin-get-order-payable-summary
```

## Authentication

Browser and frontend clients must call this function with:

- Supabase anon key in the normal Supabase function request path.
- Admin user JWT in `Authorization: Bearer <admin-jwt>`.

Frontend clients must never receive or use the service-role key.

The Edge Function may use the service-role key internally to read the protected summary RPC and adjustment rows after admin JWT and order-scope checks pass.

## Authorization

Phase 3D uses the same admin guard as the order time write functions:

| Role | Access |
|---|---|
| `owner` | Can read any order payable summary. |
| `super_admin` | Can read any order payable summary. |
| `store_manager` | Can read orders where assigned location matches pickup or return location. |
| `staff` / `part_time` | Not allowed until a future read-only permission contract is defined. |

## Request

```json
{
  "order_id": "00000000-0000-0000-0000-000000000000"
}
```

`orderId` camelCase is also accepted for client compatibility.

## Successful Response

```json
{
  "ok": true,
  "actor": {
    "role": "owner",
    "scope": "global",
    "assigned_location_id": null
  },
  "summary": {
    "order_id": "00000000-0000-0000-0000-000000000000",
    "order_number": "SMOKE-ORDER-TIME-PHASE3B-71e3133b",
    "base_rental_total": 1000,
    "base_rental_total_source": "orders.total_price",
    "order_items_subtotal_total": 1000,
    "deposit_total": 0,
    "active_fee_adjustments_total": 0,
    "active_discount_adjustments_total": 0,
    "active_adjustments_net_total": 0,
    "voided_fee_adjustments_total": 621,
    "voided_discount_adjustments_total": 0,
    "active_adjustment_count": 0,
    "voided_adjustment_count": 3,
    "final_payable_total": 1000,
    "calculation_note": "orders.total_price is base rental total; active fee adjustments are added; active discount adjustments are subtracted; voided adjustments are audit-only."
  },
  "adjustments": [
    {
      "id": "00000000-0000-0000-0000-000000000000",
      "order_id": "00000000-0000-0000-0000-000000000000",
      "adjustment_type": "late_return_fee",
      "amount": 300,
      "reason": "Smoke validation",
      "discount_eligible": false,
      "prepaid_eligible": false,
      "included_in_urent_share": false,
      "created_by": "00000000-0000-0000-0000-000000000000",
      "created_at": "2026-06-13T00:00:00.000Z",
      "voided_at": "2026-06-13T00:01:00.000Z",
      "is_active": false
    }
  ],
  "guardrails": {
    "canonical_read_path": "get_order_payable_summary",
    "frontend_service_role_required": false,
    "orders_total_price_unchanged": true,
    "order_items_unchanged": true,
    "no_urent_change": true,
    "no_prepaid_change": true,
    "voided_adjustments_excluded_from_final_payable": true
  }
}
```

## Error Response

```json
{
  "ok": false,
  "code": "ORDER_PAYABLE_SUMMARY_LOOKUP_FAILED",
  "message": "訂單應付摘要查詢失敗"
}
```

Error responses must be safe for admin UI debug drawers. They must not include service-role keys, JWTs, anon keys, raw database URLs, or stack traces.

## Calculation Contract

The API must return the canonical fields from `get_order_payable_summary`:

| Field | Rule |
|---|---|
| `base_rental_total` | Base rental total from `orders.total_price`, falling back to item subtotals only when needed. |
| `active_fee_adjustments_total` | Sum of active non-discount adjustments where `voided_at is null`. |
| `active_discount_adjustments_total` | Sum of active `discount_adjustment` rows where `voided_at is null`. |
| `final_payable_total` | `base_rental_total + active_fee_adjustments_total - active_discount_adjustments_total`. |
| `adjustments` | Full order adjustment list for admin display and audit context. |

Voided adjustments must remain visible in the adjustment list but must not affect `final_payable_total`.

## Consumer Rules

- UI must not calculate final payable total in JavaScript.
- UI must not label `orders.total_price` as final payable.
- UI should label `orders.total_price` or `base_rental_total` as base rental total.
- UI should show `final_payable_total` from this API as the payable total.
- UI may show voided adjustments only in history/audit sections.
- LINE Bot, AI Agent, Accounting exports, and Admin must use this API or a server-side equivalent that reads the same RPC.

## Non-Goals

Phase 3D does not:

- Rewrite `orders.total_price`.
- Recalculate `order_items.days` or `order_items.subtotal`.
- Change Urent revenue-share logic.
- Change prepaid/stored-value deduction logic.
- Add customer-facing payment capture for adjustments.
