# Readdy Admin Profile Hardening Required Sync

Canonical commit hash:

```text
6fa77517cab95c1c4866e8e91e125ac4c4c69dbf
```

This document is the public Readdy handoff for LensBank admin invitation and admin profile hardening. It contains no secrets, no production keys, and no private deployment details.

## Canonical Security Contract

GitHub canonical commit `6fa77517cab95c1c4866e8e91e125ac4c4c69dbf` is the required source contract for admin invitation and admin profile mutation behavior.

Readdy must not overwrite or regress the following files without Codex review:

- `supabase/functions/accept-admin-invitation/index.ts`
- `supabase/functions/update-admin-profile/index.ts`
- `supabase/functions/deactivate-admin-profile/index.ts`
- `readdy-frontend/supabase/functions/accept-admin-invitation/index.ts`
- `readdy-frontend/supabase/functions/update-admin-profile/index.ts`
- `readdy-frontend/supabase/functions/deactivate-admin-profile/index.ts`
- `readdy-frontend/src/pages/admin/admins/components/AdminModals.tsx`

## Required Checks Before Editing `/admin/admins`

Before Readdy changes `/admin/admins`, admin invitation, or admin profile edit/deactivate UI, confirm all of the following remain true:

- No `localStorage` role fallback is used for permission decisions.
- No frontend direct `admin_profiles.update()` or `admin_profiles.delete()` exists.
- `update-admin-profile` does not accept `status` changes.
- `deactivate-admin-profile` is owner-only.
- `accept-admin-invitation` returns `ROLE_INVALID` for unknown invitation roles.
- Local roles must have a valid existing location:
  - `store_manager`
  - `staff`
  - `part_time`

## Sync Instruction For Readdy

Please read this document before making any admin invitation or admin profile changes.

Do not redo the `package.json` dependency fix in Readdy sandbox. That fix is owned by the canonical/private repo pipeline.

Do not directly Publish from Readdy sandbox.

If changing admin invitation or admin profile behavior, preserve the security contract from canonical commit `6fa7751`:

- canonical role mapping must remain intact
- unknown invite roles must not fall back to `staff`
- update flow must not change `status`
- deactivate flow must remain owner-only
- local-role location validation must remain enforced by backend code
- frontend must call Edge Functions for admin profile mutation

Any proposed Readdy change in this area must be exported and reviewed by Codex before it can enter the canonical release path.
