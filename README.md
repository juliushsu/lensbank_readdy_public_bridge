# LensBank Readdy Public Bridge

This repository is the public handoff bridge for Readdy-facing LensBank work.

It is not the production source of truth. Production code, Edge Functions, database migrations, environment files, deployment commands, private endpoint maps, and full schema dumps must not be placed here.

## Purpose

- Publish Readdy-readable governance rules.
- Publish sanitized Codex audit summaries.
- Publish UI and workflow handoff contracts for Readdy alignment.

## Canonical Rule

The private production repository remains the canonical source for deployable frontend code, Edge Functions, migrations, and security patches. Readdy must not publish production directly from a sandbox package when a change touches auth, permissions, admin, scheduling, inventory, customer, order, payment, LINE binding, AI agent, Edge Functions, RLS, or Supabase schema.

## Public Safety Boundary

Allowed content:

- Sanitized governance documents.
- Public RBAC summaries.
- Readdy prompt and handoff contracts.
- UI acceptance criteria.
- Codex audit summaries without implementation internals.

Forbidden content:

- Production source code.
- Secrets, tokens, env files, anon keys, service keys, or private project refs.
- Full DB schema dumps or SQL apply/rollback scripts.
- Internal file system paths.
- Private deployment details.
- Edge Function implementation details.

