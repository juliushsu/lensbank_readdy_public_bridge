# Start Here: Readdy Order Time UI Sandbox

Status: public handoff entrypoint
Date: 2026-06-13

## Project Background

LensBank is adding order pickup/return time support in a staged, contract-first way.

The backend Staging foundation is already prepared for:

- customer requested pickup/return time;
- admin confirmed pickup/return time;
- order fee adjustments;
- canonical payable summary;
- read-only payable summary API.

This handoff is for Readdy UI Sandbox only. It is not a runtime merge, production deploy, database migration, or Cloudflare deploy.

Readdy is not responsible for proving Supabase Branch `ydubnjompnybshscosfd` connectivity. Codex owns Supabase Branch target verification and Staging runtime validation after the Readdy export is reviewed.

## Required Reading Order

Read these files in order before making UI changes:

1. [README_FIRST_FOR_READDY.md](./README_FIRST_FOR_READDY.md)
2. [ORDER_TIME_FILE_INDEX.md](./ORDER_TIME_FILE_INDEX.md)
3. [ORDER_TIME_UI_IMPLEMENTATION_GUIDE.md](./ORDER_TIME_UI_IMPLEMENTATION_GUIDE.md)
4. [PHASE4A_ORDER_TIME_UI_SANDBOX_PROMPT.md](./PHASE4A_ORDER_TIME_UI_SANDBOX_PROMPT.md)
5. [ORDER_PAYABLE_READ_API_CONTRACT.md](../contracts/ORDER_PAYABLE_READ_API_CONTRACT.md)

## Allowed This Phase

Readdy may work on:

- Checkout Requested Time UI
- Admin Requested/Confirmed Time UI
- Read-Only Payable Summary UI

Target routes:

- `/account/checkout`
- `/admin/orders/:id`

## Forbidden This Phase

Do not:

- modify rental calculation.
- modify payable calculation.
- modify Urent.
- modify prepaid/stored-value behavior.
- modify DB schema.
- modify migrations.
- modify Supabase Edge Functions.
- add fee adjustment write UI.
- calculate final payable in frontend code.
- publish production.
- deploy Cloudflare.
- deploy Supabase.

## Required Completion Output

When the UI sandbox is complete, provide:

- modified file list;
- build result;
- ZIP export;
- known limitations or follow-up notes.

Do not publish from Readdy. Export only.

Do not report Supabase Branch `ydubnjompnybshscosfd` runtime validation as complete; Codex will run that gate.
