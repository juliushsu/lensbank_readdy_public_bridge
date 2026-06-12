# P0-G5B.2 Identity Upload MVP Acceptance Checklist

Date: 2026-06-11

Scope: Acceptance checklist only. No implementation.

## Release Boundary

MVP acceptance applies only to:

```text
canonical preview / RC
```

It does not authorize:

- production deploy;
- Readdy production direct patch;
- admin sidebar/admin shell replacement;
- real identity document upload;
- DB migration;
- DDL/DML;
- seed data.

## A. Route Acceptance

| Check | Expected |
|---|---|
| `/account/identity` exists | Authenticated customer can open route |
| `/account/identity?orderId=...` exists | Route reads query and shows order context when available |
| Unauthenticated access | Redirects through login without blank screen |
| Verified status | Shows completed state, no upload warning |
| Pending status | Shows waiting review state |
| Missing status | Shows front/back upload form |
| Rejected status | Shows reupload path |
| Status unavailable | Shows fallback and does not break page |

## B. Customer Page Acceptance

### Checkout Success

| Check | Expected |
|---|---|
| Order success still visible | Yes |
| Existing `查看我的訂單` remains | Yes |
| Existing browse CTA remains | Yes |
| New identity CTA | `完成取件驗證` |
| CTA target with order id | `/account/identity?orderId={orderId}` |
| CTA target without order id | `/account/identity` |
| Gateway failure | Does not hide order success |

### Account Orders

| Check | Expected |
|---|---|
| Orders still load from Supabase | Yes |
| Identity status request fires | `GET /identity/me/status` |
| Missing/rejected state | Notice CTA to `/account/identity` |
| Pending state | Notice explains waiting review |
| Verified state | No alarming warning |
| Gateway failure | Orders remain visible |

### Account Order Detail

| Check | Expected |
|---|---|
| Order detail still loads | Yes |
| Identity status request fires | `GET /identity/me/status` |
| Missing/rejected CTA | `/account/identity?orderId={orderId}` |
| Pending state | Waiting review |
| Verified state | No workflow disruption |
| Gateway failure | Order detail remains usable |

## C. Gateway Wiring Acceptance

| Check | Expected |
|---|---|
| Status method | `getMyIdentityStatus` calls `/identity/me/status` |
| Upload session method | `createIdentityUploadSession` calls `/identity/upload-session` |
| Upload complete method | `completeIdentityUpload` calls `/identity/upload-complete` |
| Idempotency key | Present for mutation requests |
| Auth header | Present when session exists |
| Service role | Never in frontend |
| Signed URL / upload target logging | Not logged |
| Storage object path logging | Not logged |
| Raw token logging | Not logged |

## D. Upload UX Acceptance

| Check | Expected |
|---|---|
| Front file required | Yes |
| Back file required | Yes |
| Submit disabled until both sides valid | Yes |
| MIME validation | JPEG/PNG/WebP or gateway-approved list |
| Size validation | Matches gateway-approved max |
| Preview | Local preview only, no remote private path displayed |
| Replace file | Supported before submit |
| Upload progress | Visible enough for slow network |
| Upload error | Recoverable retry |
| Expired session | Can restart upload session |
| Success state | `已送出，等待門市確認` |

## E. Fallback / Failure Acceptance

| Scenario | Expected UI |
|---|---|
| Missing gateway base URL | Non-blocking fallback |
| 401/403 status | Non-blocking fallback; prompt login/support only if appropriate |
| Gateway 404 | Non-blocking fallback unless contract maps to missing |
| Gateway 5xx | Non-blocking fallback and retry |
| CORS failure | Non-blocking fallback and diagnostic without secrets |
| Upload-session fails | Stay on upload page, allow retry |
| One side upload fails | Do not call upload-complete; allow retry |
| upload-complete fails | Keep page recoverable; show retry/support |

## F. Security Acceptance

Must not expose:

- service-role keys;
- Supabase JWTs;
- signed URLs;
- upload targets;
- storage object paths;
- base64 identity images;
- customer phone/email/name in diagnostics;
- real identity image data in logs.

Must preserve:

- private bucket access only through gateway;
- short-lived upload targets;
- browser-only public anon key;
- customer self-scope;
- admin review separation.

## G. Preview Smoke Acceptance

Minimum preview smoke:

| Flow | Expected |
|---|---|
| Missing customer opens checkout success | Sees identity CTA |
| Missing customer clicks CTA | Reaches `/account/identity?orderId=...` |
| Status endpoint unavailable | Page remains usable |
| Missing customer opens account orders | Notice visible |
| Missing customer opens order detail | Notice visible with order-specific CTA |
| Verified customer opens orders | No unnecessary warning |
| Admin shell/sidebar | Unchanged |
| Production domain | Unchanged |

## H. Exit Criteria For Production Consideration

Production consideration remains blocked until:

1. Readdy production parity/preservation is approved.
2. Identity gateway environment target is production-safe.
3. Authenticated status reads return expected results for customer and admin.
4. Fake-document upload smoke passes in approved non-production/staging-safe environment.
5. Admin review path is available or owner-approved temporary SOP exists.
6. Owner approves production release.
