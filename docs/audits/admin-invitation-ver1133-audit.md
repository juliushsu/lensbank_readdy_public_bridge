# Admin Invitation Ver1133 Audit

Date: 2026-06-02

Scope: Readdy Ver1133 admin invitation and admin management UI alignment.

## Verdict

Status: FAIL

Ver1133 is not approved for direct Readdy Publish and is not approved for full merge into the private production repository.

## What Improved

- The admin page no longer uses browser localStorage as a role fallback for `/admin/admins`.
- `currentUserRole` is derived from trusted runtime sources:
  - `useAdminCheck().role`
  - authenticated `admin_profiles` lookup
  - lowest privilege fallback when trusted role loading fails
- The invitation button is driven by role hierarchy helper output instead of a hardcoded owner-only condition.
- The edit button is now conditionally rendered through an edit guard.
- Invite payload now includes operational staff fields for staff and part-time invitations.

## Blocking Findings

### 1. Package dependency remains unresolved

`eslint-config-airbnb-typescript` is still present in the package manifest. The package cannot be installed with the current dependency declaration, so build validation cannot pass.

Impact: Ver1133 cannot be considered a deployable package.

Required Readdy action:

- Remove the unresolved dependency if it is unused.
- Do not add lint scripts that depend on unavailable packages.
- Ensure install and build both pass before export.

### 2. Admin profile edit/delete still uses direct client writes

The admin edit modal still performs direct updates to `admin_profiles`.

The admin delete modal still performs direct deletes from `admin_profiles`.

Impact: This conflicts with the LensBank security direction that admin profile mutation must be enforced by backend hierarchy checks, not by browser-only guards.

Required Readdy action:

- Do not implement direct client update/delete to admin profile tables.
- Use the approved backend mutation path from the private production repository.
- Do not hard delete admin profiles.

### 3. Store manager edit guard conflicts with admin permissions

The edit guard is derived from the same helper used for invite/manageable roles. That means store managers may be treated as able to edit staff or part-time admins in UI logic.

Current governance expectation:

- Owner can edit admins according to backend owner protections.
- Super admin can edit permitted non-owner roles.
- Store manager currently must not edit or deactivate admin profiles.
- Store manager may invite staff or part-time only if backend/RLS also enforces this safely.

Required Readdy action:

- Separate invite roles from edit roles.
- Keep store manager edit and deactivate hidden unless the private canonical backend explicitly enables it.

### 4. Invite creation is still a direct client insert

The invite modal still inserts directly into the admin invitation table from the frontend.

Impact: This may be acceptable only if RLS strictly enforces caller role, target role, assigned location, status, and invite scope. Otherwise it should be moved to a backend endpoint.

Required Readdy action:

- Treat this as pending security review.
- Do not assume frontend role filtering is sufficient.

## Merge Recommendation

Do not merge Ver1133 as a package.

Selective salvage candidates:

- Keep the removal of localStorage role fallback.
- Keep trusted role source ordering.
- Keep the invitation button visibility logic if backend authorization remains canonical.

Do not salvage:

- Direct admin profile update/delete.
- Direct admin invitation insert unless approved RLS/backend enforcement is confirmed.
- Any production deployment configuration.

