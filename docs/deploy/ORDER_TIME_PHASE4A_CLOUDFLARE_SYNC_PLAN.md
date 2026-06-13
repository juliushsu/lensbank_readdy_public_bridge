# Order Time Phase 4A Cloudflare Sync Plan

Status: planning only
Date: 2026-06-13

## Purpose

Define a safe frontend synchronization path for the Order Time UI after Staging DB/API foundations are ready.

This document is a plan only. Do not deploy production, do not change production/main, and do not publish Readdy to production.

## Current Frontend Divergence

Known bundle divergence from the Order Time deployment audit:

| Surface | Observed bundle |
|---|---|
| custom-domain production / Readdy preview | `index-DQlL2sci.js` |
| Cloudflare Pages | `index-CzFheSak.js` |
| local source build | `index-BkyusEtC.js` |

Implications:

- The custom domain, Readdy preview, Cloudflare Pages, and local source are not proven to be the same frontend version.
- Authenticated admin route parity cannot be inferred from public SPA HTML alone.
- Order Time UI must not be pushed from any single divergent surface directly into production.

## Canonical Source Policy

Future formal frontend releases should use:

```text
GitHub canonical source
-> Cloudflare Pages preview
-> owner acceptance
-> approved main / production deploy
```

Readdy role:

```text
Readdy = UI sandbox only
```

Readdy must not be used as the production release source for:

- checkout
- admin orders
- payable summary
- orders
- pricing
- auth
- permissions
- Edge Function integration
- Supabase schema/RLS behavior

## Phase 4A UI Flow

Recommended flow:

1. Give Readdy the Phase 4A UI sandbox prompt.
2. Readdy creates sandbox UI only.
3. Readdy exports a ZIP/source package.
4. Codex reviews the ZIP against GitHub canonical source.
5. Codex ports acceptable changes into a GitHub canonical staging branch.
6. Codex verifies no forbidden logic was introduced:
   - no frontend final payable calculation;
   - no direct adjustment write UI;
   - no Urent/prepaid/rental calculation change;
   - no direct core order pricing mutation.
7. Run frontend build from GitHub canonical branch.
8. Deploy Cloudflare Pages preview only.
9. Codex verifies Cloudflare preview environment targets Supabase Branch `ydubnjompnybshscosfd`.
10. Run staging preview smoke.
11. Owner reviews Cloudflare preview.
12. Only after explicit owner approval, merge/release to main/production.

## Cloudflare Pages Preview Strategy

Cloudflare Pages should deploy only from the GitHub canonical branch.

Preview deploy requirements:

- Build command and output directory match the canonical repo.
- Environment variables point to Staging Supabase branch for preview testing; this is verified by Codex, not Readdy.
- No production Supabase DB writes are part of preview validation.
- Preview URL is recorded in the release report.
- Production custom domain remains unchanged.

Do not:

- manually upload Readdy ZIP to Cloudflare production.
- connect Readdy preview as a production target.
- overwrite the current custom domain from Readdy.
- merge to main before preview smoke and owner acceptance.

## Suggested Branching

Suggested branch names:

```text
phase/order-time-phase4a-ui-sandbox-intake
phase/order-time-phase4b-frontend-staging-preview
```

The Readdy export should be treated as input evidence, not as the deployable artifact.

## Safe Production Cutover Sequence

Only after owner approval:

1. Freeze Readdy publish for the release window.
2. Record current production bundle fingerprint.
3. Record current Cloudflare Pages bundle fingerprint.
4. Confirm GitHub branch/commit to release.
5. Confirm Supabase production DB migration/apply approval separately if needed.
6. Confirm production Edge Functions are deployed only through approved production deploy gate.
7. Merge the approved branch to main.
8. Let Cloudflare Pages build from main.
9. Run production smoke with a non-destructive checklist.
10. Monitor checkout, admin order detail, auth, and order creation.

Rollback source must be GitHub canonical, not Readdy sandbox.

## Explicit Non-Actions In Phase 4A

Phase 4A does not:

- deploy production;
- deploy Cloudflare production;
- publish Readdy production;
- change DNS;
- apply production DB migrations;
- change Supabase production functions;
- change Urent, prepaid, stored-value, or rental calculation logic.
