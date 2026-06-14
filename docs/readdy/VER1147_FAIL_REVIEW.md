# Ver1147 Fail Review

Status: failed review
Date: 2026-06-14
Scope: Order Time Phase 4B.2 UX validation review

## 1. Ver1147 Passed Items

Ver1147 correctly moved the checkout UX in the right direction for closed-store days:

- Closed-day warning exists when the selected pickup or return store is not open.
- The UI tells customers to choose another date or store.
- The UI includes special-case copy explaining that store staff may assist with special arrangements.
- Submit is disabled when the selected pickup or return date has no available time slots.
- Payable Summary still follows the read-only contract and calls `admin-get-order-payable-summary`.

These are good UX additions for physical rental operations where stores may have closed days, special business days, and exceptional store-managed cases.

## 2. Ver1147 FAIL

`requestedPickupTime` and `requestedReturnTime` are not synchronized with the latest available time slots.

Failure scenario:

```text
Customer selects Date A
↓
Customer selects 18:30
↓
Customer changes to Date B
↓
18:30 does not exist in Date B's latest time slots
↓
React state still keeps 18:30
↓
Checkout can submit an illegal requested time
```

The same risk applies when the customer changes:

- `pickupLocationId`
- `returnLocationId`
- `startDate`
- `endDate`
- `pickupTimeSlots`
- `returnTimeSlots`

This violates:

- `ORDER_TIME_AND_FEE_ADJUSTMENT_CONTRACT`

The contract requires customer requested pickup/return times to be constrained by the selected pickup/return store business hours.

## 3. Required Outcome

Ver1147 must not be treated as Cloudflare Preview ready.

Ver1148 must fix state consistency before any owner acceptance or preview deployment.
