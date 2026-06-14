# Ver1148 Required Fix

Status: required before Cloudflare Preview
Date: 2026-06-14
Scope: Order Time checkout UX and build retention

## Blocker 1: State Consistency

When any of the following values change:

- `pickupLocationId`
- `returnLocationId`
- `startDate`
- `endDate`

or when either latest slot list changes:

- `pickupTimeSlots`
- `returnTimeSlots`

the selected requested time must be checked against the latest slots.

If the currently selected time does not exist in the latest slots, it must be cleared automatically:

```ts
if (requestedPickupTime && !pickupTimeSlots.includes(requestedPickupTime)) {
  setRequestedPickupTime('');
}

if (requestedReturnTime && !returnTimeSlots.includes(requestedReturnTime)) {
  setRequestedReturnTime('');
}
```

This prevents an old valid time from one date/store from being submitted after the customer changes to another date/store.

## Submit-Time Validation

Before submitting checkout, validate again:

```ts
if (requestedPickupTime && !pickupTimeSlots.includes(requestedPickupTime)) {
  block submit;
}

if (requestedReturnTime && !returnTimeSlots.includes(requestedReturnTime)) {
  block submit;
}
```

Required customer-facing result:

- Do not submit the order.
- Show a clear message asking the customer to reselect a valid pickup or return time.

## Blocker 2: Build Fix Retention

Ver1148 must retain the Ver1146 build fix.

Do not regress build dependencies.

Required:

```json
"eslint-config-airbnb-typescript": "18.0.0"
```

The package must not request the nonexistent version:

```json
"eslint-config-airbnb-typescript": "^18.2.1"
```

Ver1148 must also retain the `unplugin-auto-import` related build correction from Ver1146.

Allowed approaches:

- keep `unplugin-auto-import` dependency and ensure the Vite plugin resolves correctly; or
- remove the AutoImport plugin from `vite.config.ts` if that is the chosen build-safe path.

The final package must pass:

```bash
npm install
npm run build
```

## Non-Goals

Do not modify:

- rental fee calculation.
- rental day calculation.
- payable calculation.
- Urent revenue sharing.
- prepaid or stored-value behavior.
- Supabase schema.
- Supabase Edge Functions.
- production deployment.
