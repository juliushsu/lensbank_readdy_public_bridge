# Order Time File Index

Status: Readdy handoff index
Date: 2026-06-13

## Must Read

Readdy UI sandbox implementer must read these before changing UI:

1. [README_FIRST_FOR_READDY.md](./README_FIRST_FOR_READDY.md)
2. [ORDER_TIME_UI_IMPLEMENTATION_GUIDE.md](./ORDER_TIME_UI_IMPLEMENTATION_GUIDE.md)
3. [PHASE4A_ORDER_TIME_UI_SANDBOX_PROMPT.md](./PHASE4A_ORDER_TIME_UI_SANDBOX_PROMPT.md)
4. [ORDER_TIME_AND_FEE_ADJUSTMENT_CONTRACT.md](../contracts/ORDER_TIME_AND_FEE_ADJUSTMENT_CONTRACT.md)
5. [ORDER_PAYABLE_SUMMARY_CONTRACT.md](../contracts/ORDER_PAYABLE_SUMMARY_CONTRACT.md)
6. [ORDER_PAYABLE_READ_API_CONTRACT.md](../contracts/ORDER_PAYABLE_READ_API_CONTRACT.md)

## Reference

These files are useful for Codex review, staging verification, and release planning:

1. [ORDER_TIME_PHASE3D_PAYABLE_READ_API_REPORT.md](../reports/ORDER_TIME_PHASE3D_PAYABLE_READ_API_REPORT.md)
2. [ORDER_TIME_PHASE4A_DEPLOYMENT_GATE_CHECKLIST.md](../deploy/ORDER_TIME_PHASE4A_DEPLOYMENT_GATE_CHECKLIST.md)
3. [ORDER_TIME_PHASE4A_CLOUDFLARE_SYNC_PLAN.md](../deploy/ORDER_TIME_PHASE4A_CLOUDFLARE_SYNC_PLAN.md)

## Readdy Scope Summary

Readdy may work on:

- `/account/checkout` requested time UI.
- `/admin/orders/:id` requested/confirmed time display.
- `/admin/orders/:id` read-only payable summary display.

Readdy must not work on:

- database migrations.
- Supabase Edge Functions.
- RLS.
- Urent revenue share.
- prepaid/stored-value deduction.
- rental fee/day calculation.
- fee adjustment create/void operations.
- production deployment.

## Canonical Release Path

```text
Readdy UI sandbox
-> export ZIP/source
-> Codex review
-> GitHub canonical branch
-> build
-> Cloudflare Pages preview
-> owner acceptance
-> explicit production gate
```

The Readdy sandbox is not a production release source.
