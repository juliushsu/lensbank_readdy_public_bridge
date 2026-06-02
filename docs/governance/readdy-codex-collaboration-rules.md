# Readdy / Codex Collaboration Rules

This document defines the public collaboration rules between Readdy and Codex for LensBank.

## Source of Truth

The production source of truth is the private LensBank production repository.

This public bridge repository is only for sanitized specifications, handoff notes, and audit summaries.

Readdy must not treat this bridge repository as deployable source code.

## Readdy Must Not Publish Production Directly

Readdy sandbox output must not be published directly to production when changes touch:

- auth
- permissions
- admin
- scheduling
- inventory
- customer
- order
- payment
- LINE binding
- AI agent
- Edge Functions
- RLS
- Supabase schema

## Required Flow for Sensitive Changes

1. Readdy prepares UI or workflow changes.
2. Readdy exports a package or publishes a sanitized handoff summary.
3. Codex audits the change against the private canonical implementation.
4. Only approved selective changes are merged into the private production repository.
5. Build and security checks must pass.
6. Deployment happens from the private canonical source, not from Readdy sandbox.

## Public Bridge Content Rules

Allowed:

- sanitized RBAC summaries
- Readdy prompts
- handoff contracts
- UI acceptance criteria
- high-level audit summaries

Forbidden:

- source code from production repository
- Edge Function implementation details
- database schema dumps
- SQL migrations
- tokens, env values, keys, secrets
- private endpoint maps
- internal file paths
- production deployment commands

## Admin Management Rules

Admin role authority must come from authenticated backend state. Browser localStorage must not be used as a role fallback for admin permissions.

Admin invitation UI must follow the public invite matrix, but backend/RLS remains the real security boundary.

Admin profile update and deactivate must not be implemented as direct browser writes.

Hard delete of admin profiles is forbidden in public Readdy proposals unless the private canonical backend explicitly approves it.

## Audit Outcome Labels

PASS:

- Safe to selectively merge or use as a Readdy reference.

FAIL:

- Not safe to merge or publish as-is.

CONDITIONAL:

- May be used only after listed blockers are resolved and Codex re-audits.

