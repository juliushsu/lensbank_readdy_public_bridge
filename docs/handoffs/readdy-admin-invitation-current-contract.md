# Readdy Admin Invitation Current Contract

This is the public Readdy-facing contract for LensBank admin invitation UI work. It is a specification only, not production source code.

## Roles

LensBank admin roles:

- owner
- super_admin
- store_manager
- staff
- part_time

## Invite Role Matrix

Owner may invite:

- owner
- super_admin
- store_manager
- staff
- part_time

Super admin may invite:

- super_admin
- store_manager
- staff
- part_time

Super admin must not invite:

- owner

Store manager may invite:

- staff
- part_time

Staff and part-time may invite:

- no roles

## Trusted Role Source

The admin invitation UI must not use browser localStorage, query strings, mock role state, or arbitrary frontend state as an authority for admin role.

Allowed role sources:

1. Existing authenticated admin check hook result.
2. Authenticated admin profile lookup for the current signed-in user.
3. If both fail, default to lowest privilege and show an error.

Lowest privilege fallback:

- role: staff
- invite button hidden
- edit/deactivate controls hidden
- owner-only download controls hidden

## Invite Button Rule

The invite button may be shown only when the current trusted role has at least one allowed target role.

The button must not be owner-only, because super admins are allowed to invite permitted roles.

The button must not be controlled by localStorage.

## Invite Modal Rule

The role selector must only show roles allowed by the trusted caller role.

Required location behavior:

- owner and super_admin targets are global roles and should not require assigned location.
- store_manager, staff, and part_time require a valid assigned location unless private canonical backend rules say otherwise.

Store manager location behavior:

- Store managers must not be able to invite staff or part-time into another location unless backend/RLS explicitly verifies assigned location scope.

## Edit and Deactivate Rule

Invite capability and edit capability are not the same.

Current public contract:

- owner: may edit according to backend owner protections.
- super_admin: may edit permitted non-owner roles.
- store_manager: must not edit or deactivate admin profiles.
- staff and part_time: no admin mutation.

Do not use the invite role helper as the edit role helper unless it exactly matches this matrix.

## Backend Boundary

Frontend role filtering improves usability, but it is not a security boundary.

Admin invitation creation must be protected by one of:

- backend endpoint with authenticated hierarchy enforcement, or
- strict RLS policies that enforce caller role, target role, status, assigned location, and invite scope.

Admin profile update/deactivate must use the private canonical backend mutation path. Readdy must not introduce direct admin profile update/delete from browser code.

