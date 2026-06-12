# P0-G5B.7 Readdy Handoff Package - Identity Upload MVP

Date: 2026-06-12

Purpose:

- Give Readdy a precise, narrow handoff for porting only the customer identity upload MVP.
- Preserve `lensbank.com.tw` production admin shell/sidebar and all unrelated production-critical flows.
- Keep upload mutation disabled by default through dry-run.

Scope boundary:

- Handoff package only.
- No deploy.
- No DB/migration/DDL/DML.
- No production mutation.
- No Readdy production change by Codex.
- No real upload.
- No signed URL or token output.

## GitHub Public Bridge Links For Readdy

Use GitHub public links as the single source of truth for Readdy review. Do not send local filesystem paths as the handoff reference.

Public bridge repo:

```text
https://github.com/juliushsu/lensbank_readdy_public_bridge
```

Public bridge branch for this handoff:

```text
main
```

Important handoff docs:

| Purpose | GitHub Link |
|---|---|
| Readdy handoff package | [P0_G5B7_READDY_HANDOFF_PACKAGE.md](https://github.com/juliushsu/lensbank_readdy_public_bridge/blob/main/docs/audits/p0-order-lead-safety-hardening-20260611/P0_G5B7_READDY_HANDOFF_PACKAGE.md) |
| Paste-ready Readdy prompt | [READDY_IDENTITY_UPLOAD_MVP_PROMPT.md](https://github.com/juliushsu/lensbank_readdy_public_bridge/blob/main/docs/audits/p0-order-lead-safety-hardening-20260611/READDY_IDENTITY_UPLOAD_MVP_PROMPT.md) |
| Acceptance checklist | [IDENTITY_UPLOAD_MVP_ACCEPTANCE_CHECKLIST.md](https://github.com/juliushsu/lensbank_readdy_public_bridge/blob/main/docs/audits/p0-order-lead-safety-hardening-20260611/IDENTITY_UPLOAD_MVP_ACCEPTANCE_CHECKLIST.md) |
| P0 order/lead safety README | [README.md](https://github.com/juliushsu/lensbank_readdy_public_bridge/blob/main/docs/audits/p0-order-lead-safety-hardening-20260611/README.md) |

Important implementation reference file paths:

| Purpose | Path |
|---|---|
| Route registration | `readdy-frontend/src/router/config.tsx` |
| Identity upload page | `readdy-frontend/src/pages/account/identity/page.tsx` |
| Shared customer identity notice | `readdy-frontend/src/components/identity/CustomerIdentityNotice.tsx` |
| Feature flags | `readdy-frontend/src/modules/identity/featureFlags.ts` |
| Identity gateway service | `readdy-frontend/src/services/identityGatewayService.ts` |
| Checkout success modal CTA | `readdy-frontend/src/pages/account/checkout/page.tsx` |
| Checkout success route CTA | `readdy-frontend/src/pages/account/checkout/success/page.tsx` |
| Account orders CTA | `readdy-frontend/src/pages/account/orders/page.tsx` |
| Account order detail CTA | `readdy-frontend/src/pages/account/orders/detail/page.tsx` |

Visibility note:

- This public bridge intentionally contains docs only.
- It does not publish source code, env values, tokens, signed URLs, or DB credentials.
- Readdy should implement in its own production codebase using the file paths and contracts described here.

## A. Readdy Scope Boundary

### Readdy May Do Only These Items

1. Add an authenticated customer route:

```text
/account/identity
/account/identity?orderId=...
```

2. Add checkout success CTA:

```text
完成取件驗證
```

Target:

```text
/account/identity?orderId={orderId}
```

3. Add account orders missing-identity CTA:

```text
/account/identity?orderId={orderId}
```

4. Add account order detail missing-identity CTA:

```text
/account/identity?orderId={orderId}
```

5. Connect to the existing identity gateway contract:

```text
GET  /identity/me/status
POST /identity/upload-session
POST /identity/upload-complete
```

6. Protect all UI and mutation behavior with feature flags:

```text
VITE_ENABLE_IDENTITY_UPLOAD_MVP
VITE_IDENTITY_UPLOAD_MVP_DRY_RUN
```

7. Keep `VITE_IDENTITY_UPLOAD_MVP_DRY_RUN=true` by default.

8. Keep fallback non-blocking:

```text
身份資料狀態暫時無法確認，您仍可稍後再試
```

9. Preserve current production customer shopping and account behavior unless directly wiring the identity CTA.

### Readdy Must Not Do These Items

Do not modify:

- Admin sidebar.
- Admin shell.
- Admin dashboard navigation.
- Admin CMS/frontend image management.
- LIFF routes.
- LINE binding routes.
- Attendance routes.
- Frontend images management.
- Storefront sort.
- Order status schema.
- Existing order lifecycle semantics.
- Supabase database schema.
- Supabase RLS.
- Supabase migrations.
- Storage bucket configuration.
- Production Cloudflare/Readdy deployment settings.

Do not:

- Add new DB tables.
- Alter existing DB tables.
- Change Supabase RLS policies.
- Enable real production upload mutation.
- Process real identity documents during implementation.
- Expose signed URLs, upload targets, tokens, cookies, or Authorization headers.
- Rebuild or replace the whole site.
- Port Codex/pages.dev admin shell over `lensbank.com.tw`.

## B. Files And Logic To Port

Canonical/Codex preview source contains the implementation reference, but this public bridge publishes docs only. Readdy should use the path list and behavior contract below, then implement in its own production codebase without reading private source links.

### Route

| Source File Path | Logic |
|---|---|
| `readdy-frontend/src/router/config.tsx` | Adds `/account/identity` under `AuthGuard` |
| `readdy-frontend/src/pages/account/identity/page.tsx` | Dedicated customer identity upload MVP page |

Route behavior:

- Requires customer login.
- Supports optional `orderId` query.
- Returns to `/account/orders/:orderId` when `orderId` exists.
- Returns to `/account/orders` when no `orderId` exists.
- Shows safe disabled page when `VITE_ENABLE_IDENTITY_UPLOAD_MVP` is false.

### Service Wiring

| Source File Path | Methods |
|---|---|
| `readdy-frontend/src/services/identityGatewayService.ts` | `getMyIdentityStatus()` |
| `readdy-frontend/src/services/identityGatewayService.ts` | `createIdentityUploadSession()` |
| `readdy-frontend/src/services/identityGatewayService.ts` | `uploadIdentityDocumentSide()` |
| `readdy-frontend/src/services/identityGatewayService.ts` | `completeIdentityUpload()` |

Porting notes:

- `getMyIdentityStatus()` calls `GET /identity/me/status`.
- `createIdentityUploadSession()` calls `POST /identity/upload-session`.
- `completeIdentityUpload()` calls `POST /identity/upload-complete`.
- Direct upload must not run when dry-run is true.
- Do not log signed upload URLs or storage paths.
- Do not expose response tokens in UI or console.

### Feature Flags

| Source File Path | Logic |
|---|---|
| `readdy-frontend/src/modules/identity/featureFlags.ts` | `isIdentityUploadMvpEnabled()` |
| `readdy-frontend/src/modules/identity/featureFlags.ts` | `isIdentityUploadMutationDryRun()` |

Required behavior:

```text
VITE_ENABLE_IDENTITY_UPLOAD_MVP=true
```

enables customer identity upload UI.

```text
VITE_IDENTITY_UPLOAD_MVP_DRY_RUN=true
```

prevents upload-session, upload-complete, and file upload.

### Customer Flow CTA Wiring

| Source File Path | Logic |
|---|---|
| `readdy-frontend/src/pages/account/checkout/page.tsx` | Adds success modal CTA to `/account/identity?orderId={createdOrderId}` |
| `readdy-frontend/src/pages/account/checkout/success/page.tsx` | Adds success route CTA to `/account/identity?orderId={orderId}` |
| `readdy-frontend/src/pages/account/orders/page.tsx` | Adds compact identity notice and CTA |
| `readdy-frontend/src/pages/account/orders/detail/page.tsx` | Adds identity notice and order-specific CTA |
| `readdy-frontend/src/components/identity/CustomerIdentityNotice.tsx` | Shared identity status notice and CTA |

CTA text:

```text
完成取件驗證
```

CTA target:

```text
/account/identity?orderId={orderId}
```

Fallback target:

```text
/account/identity
```

### Page UX Logic

Port these UI behaviors:

- Show current identity status.
- Show Missing / Pending / Verified / Rejected / Fallback states.
- Show trust copy.
- Show front ID upload control.
- Show back ID upload control.
- Show JPG/PNG/WebP and 8MB file hint.
- Disable submit until both front and back files are selected and valid.
- In dry-run mode, clicking submit should show local success state only.
- Show `已送出審核` after dry-run submit.
- Provide `返回訂單`.

### Fallback Behavior

If identity gateway status fails:

- Do not white-screen.
- Do not block orders list.
- Do not block order detail.
- Show fallback text.
- Keep customer able to return to order or contact support.

Required fallback copy:

```text
身份資料狀態暫時無法確認，您仍可稍後再試
```

## C. UX Copy Final Draft

Use this draft as the target Readdy copy unless owner changes it in review.

### Page Title

```text
完成取件驗證
```

### Main Trust Copy

```text
為了讓您取件更快速、更安心，LensBank 採用數位身份驗證方式。
```

### No Physical ID Retention Copy

```text
完成驗證後，取件時僅需攜帶本人證件到店核對，無須將身分證件留置於店面保管。
```

### Data Use Copy

```text
您提供的資料僅用於身份驗證、電子契約簽署與租借紀錄管理。系統將依權限進行存取管理。
```

### Upload Copy

```text
身分證正面
請上傳清楚、完整、未裁切的正面照片。

身分證反面
請上傳清楚、完整、未裁切的反面照片。

支援 JPG、PNG、WebP，單一檔案上限 8MB。請確認照片清晰且四角完整，避免影響門市審核。
```

### CTA Copy

Primary entry CTA:

```text
完成取件驗證
```

Upload submit:

```text
送出審核
```

Secondary:

```text
返回訂單
```

### Success Copy

```text
已送出審核
我們已收到您上傳的身份資料，門市會接續確認。取件時請攜帶本人證件到店核對。
```

### Status Copy

| State | Title | Body |
|---|---|---|
| Missing | `完成取件驗證` | `請上傳身分證正反面，讓門市可以完成租借審核與取件安排。` |
| Pending | `身份資料已送出，等待門市確認` | `門市正在確認您的身份資料。若取件日期接近，請留意門市後續通知。` |
| Verified | `身份資料已完成驗證` | `您的身份資料已完成驗證。取件時請攜帶本人證件到店核對。` |
| Rejected | `身份資料需重新補件` | `請重新上傳清楚、完整、未裁切的身分證正面與反面照片。` |
| Fallback | `身份資料狀態暫時無法確認` | `身份資料狀態暫時無法確認，您仍可稍後再試。若取件日期接近，也可以先聯絡門市確認補件方式。` |

### Avoid These Phrases

Do not use:

- `立即完成身份驗證`
- `一般員工無法查看`
- Broad legal promises that are not verified by CTO.
- Fear-based or punitive language.
- Copy implying the order is cancelled immediately if the customer does not upload.
- Copy implying upload is already a production-live storage flow while dry-run is enabled.

## D. Readdy Implementation Prompt

The paste-ready prompt lives in:

[READDY_IDENTITY_UPLOAD_MVP_PROMPT.md](https://github.com/juliushsu/lensbank_readdy_public_bridge/blob/main/docs/audits/p0-order-lead-safety-hardening-20260611/READDY_IDENTITY_UPLOAD_MVP_PROMPT.md)

Use that file as the exact handoff prompt to Readdy.

## E. Acceptance Checklist

Owner should verify after Readdy completes the port:

| Item | Expected Result |
|---|---|
| Checkout success has `完成取件驗證` | Visible only when `VITE_ENABLE_IDENTITY_UPLOAD_MVP=true` |
| Checkout success CTA target | `/account/identity?orderId=...` |
| `/account/identity` requires login | Unauthenticated users are redirected/guarded |
| `/account/identity?orderId=...` renders after customer login | Page loads without white screen |
| Trust copy is correct | Uses approved final draft |
| No physical ID retention copy appears | `無須將身分證件留置於店面保管` |
| Data use copy appears | Identity verification, e-contract signing, rental record management |
| Front/back upload UI appears | Both controls visible |
| Submit disabled before both files | Cannot submit with missing side |
| Dry-run submit works | Shows `已送出審核` locally |
| No upload-session in dry-run | Network must not show `/identity/upload-session` |
| No upload-complete in dry-run | Network must not show `/identity/upload-complete` |
| No storage upload in dry-run | Network must not show storage upload or signed URL request |
| Identity status request acceptable | `GET /identity/me/status` may appear |
| Account orders CTA appears | Missing/rejected state links to `/account/identity?orderId=...` |
| Order detail CTA appears | Missing/rejected state links to `/account/identity?orderId=...` |
| Admin sidebar unchanged | Production admin shell/sidebar must match current Readdy production |
| Admin shell unchanged | No Codex/pages.dev admin shell overwrite |
| LIFF unchanged | Existing LIFF routes still work |
| Attendance unchanged | Attendance routes still work |
| LINE binding unchanged | LINE binding routes still work |
| Frontend images unchanged | Existing image management remains intact |
| Storefront sort unchanged | Existing storefront sort remains intact |
| Build passes | Readdy reports command and result |

## Production Gate

Do not enable production upload mutation until all are true:

1. Owner authenticated dry-run smoke passes.
2. Readdy UI wording review passes.
3. CTO approves access-control and data-use copy.
4. Admin shell/sidebar parity is confirmed.
5. `VITE_IDENTITY_UPLOAD_MVP_DRY_RUN=true` dry-run network shows no upload-session/upload-complete/storage upload.
6. Owner explicitly approves a staging fake upload mutation test.
7. Staging fake upload mutation test passes with fake/non-real files.
8. Production release plan is approved separately.

## Handoff Summary

Recommended Readdy action:

- Port only the customer identity upload MVP.
- Preserve current `lensbank.com.tw` production admin and operational features.
- Keep dry-run on.
- Report modified files, build result, smoke result, and any deviations from this scope.
