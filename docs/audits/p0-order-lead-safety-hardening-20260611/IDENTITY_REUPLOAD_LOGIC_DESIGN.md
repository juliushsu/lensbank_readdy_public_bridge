# Identity Re-upload Logic Design

Date: 2026-06-12

Source package reviewed:

```text
/Users/chishenhsu/Downloads/LensBank-Ver1141/
```

Scope:

- Design / audit only.
- No deploy, DB change, migration, DDL/DML, production mutation, or source change.
- No real identity upload.
- No secret, token, signed URL, storage path, or personal data output.

## Executive Recommendation

LensBank should support verified customer re-upload, but not as a silent overwrite.

Recommended safe model:

- Keep the currently verified document active until the replacement is approved.
- Treat a customer re-upload as a new pending review version.
- Admin should see whether the upload is first upload, rejected re-upload, pending replacement, or verified-document update.
- If the current backend cannot version documents safely, Readdy preview may only show UI/dry-run behavior and must not claim real update upload is live.

This avoids the highest operational risk: a previously verified customer becoming blocked immediately after uploading a replacement that has not been reviewed.

## A. Current Identity Status Model

Ver1141 customer UI currently maps gateway statuses into these page states:

| Current state | Source meaning | Current customer UI behavior |
|---|---|---|
| `missing` | `overallStatus = not_uploaded` | `/account/identity` shows upload form for front/back identity document. Account notices can show補件 CTA when feature flag is on. |
| `fallback` | Status unknown, gateway failure, disabled state, or unsupported status | Ver1141 allows upload form with warning: status cannot be confirmed but customer may upload first. |
| `pending` | Uploaded and waiting for review | `/account/identity` shows status card: already submitted, waiting for store confirmation. Upload form is not shown. |
| `verified` | Identity accepted / ready | `/account/identity` shows verified status card. Upload form is not shown. `CustomerIdentityNotice` hides itself when verified. |
| `rejected` | Admin rejected or requested correction | `/account/identity` shows upload form and rejected warning. |
| `submitted` | Local dry-run success after submit | `/account/identity` shows "已送出審核"; not durable DB status. |

Important current limitation:

- `pending` and `verified` are non-upload states in Ver1141.
- This means the UI does not yet support "replace pending photo" or "verified customer update document".
- `submitted` is local page state only; it is not evidence of successful backend upload.

## B. Re-upload Use Cases

### 1. Verified customer wants to update document

Recommended behavior:

- Show verified status.
- Offer secondary CTA: `更新證件資料`.
- Explain that the existing verified status remains valid until the update is reviewed.
- On submit, create a replacement pending version.
- Admin reviews the replacement.
- After approval, new version becomes active verified document.

Risk note:

- Do not immediately invalidate the previous verified state unless law/operations explicitly require it.
- Immediate invalidation can block pickup or contract generation even though the customer was previously verified.

### 2. Pending customer wants to replace uploaded document

Recommended behavior:

- Show pending status.
- Offer CTA: `重新上傳`.
- Explain that the newest submitted version will be reviewed.
- On submit, mark/record this as a pending replacement.
- Admin should see newest upload first and be warned that it replaces a pending version.

### 3. Rejected customer must re-upload

Recommended behavior:

- Show rejected state as action-required.
- CTA: `重新上傳證件`.
- Explain reason if backend provides a non-sensitive rejection reason.
- On submit, transition back to pending review.

### 4. Fallback status but customer wants to upload

Recommended behavior:

- Keep the page non-blocking.
- Show warning: status temporarily unavailable.
- Allow CTA: `先上傳證件資料`.
- In dry-run preview, show local submitted state only.
- In real backend, submit should create a pending upload if auth is valid.

Risk note:

- Fallback submit should not overwrite verified state blindly.
- Gateway/backend should decide whether it is first upload, replacement, or update.

### 5. Missing customer first upload

Recommended behavior:

- Show standard upload form.
- Submit both front/back.
- Transition to pending review.
- Admin sees first upload.

## C. Status Transition Recommendation

### Minimal model without new durable DB status

If LensBank wants to avoid DB migration now, use existing durable statuses:

| Current | Action | Recommended durable status | UI label |
|---|---|---|---|
| missing | first upload | pending | 待門市確認 |
| rejected | re-upload | pending | 已重新送出，待門市確認 |
| pending | replacement upload | pending | 已重新送出，將以最新版本為準 |
| verified | update upload | pending for new version, keep current verified active | 更新資料待確認 |
| fallback | upload attempt | pending if backend accepts; otherwise local fallback_submitted in preview | 已送出 / 狀態待確認 |

This can work if the backend can store multiple rows/versions and determine the active document. It does not require a new public UI status, but it does require backend versioning or equivalent semantics.

### Preferred metadata later

If backend changes are later approved, add semantic metadata rather than overloading status:

| Field concept | Purpose |
|---|---|
| `upload_intent` | `first_upload`, `reupload_rejected`, `replace_pending`, `update_verified`, `staff_assist` |
| `version_group_id` | Groups front/back versions as one submission |
| `replaces_document_id` | Links replacement to old document |
| `active_after_approval` | Replacement becomes active only after approval |
| `superseded_at` | Old document retired after approval |
| `review_required_reason` | Reason shown to admin |

### Should LensBank add `pending_update`?

Not required for MVP if backend/admin can label upload intent.

Recommended:

- Use existing `pending` as durable review status.
- Add UI/admin wording "更新資料待確認" when upload intent is `update_verified`.
- Avoid adding a new durable status until backend/admin/reporting needs are clearer.

## D. Old Document Handling

### Do not overwrite old files in place

Recommendation:

- New upload should create new storage objects and new document rows.
- Do not reuse the same storage path.
- Do not replace the file behind an existing verified record.

Reason:

- Overwrite destroys auditability.
- Admin cannot compare old/new.
- Contract and historical rental records may reference the old document.

### Keep old verified document active until replacement approval

Recommended lifecycle:

1. Customer with verified status starts update.
2. New front/back upload creates pending replacement version.
3. Old verified version remains active and identity-ready.
4. Admin approves replacement.
5. New version becomes active verified.
6. Old version becomes superseded/masked/archived according to retention policy.

Lower-risk policy:

- Preserve operational continuity.
- Avoid blocking pickup for previously verified customers.
- Still gives staff a review queue for the updated document.

### Pending replacement handling

For pending customer replacement:

- The newest submission should be the review target.
- Older pending version can be marked superseded or ignored by admin UI.
- Admin should see that this is a replacement so they do not review stale images.

### Audit log

Strongly recommended for real implementation:

- `identity_upload_created`
- `identity_upload_replaced`
- `identity_update_requested`
- `identity_update_approved`
- `identity_update_rejected`
- actor type: `customer_self` / `staff_assist` / `admin`
- non-sensitive metadata only; no file path or signed URL in logs.

### Admin visibility

Admin should see:

- First upload
- Re-upload after rejected
- Replacement while pending
- Update from verified
- Which order/customer initiated the upload, if available
- Submitted time
- Whether old verified status remains active

## E. Admin Review Impact

### Should admin know first upload vs re-upload vs update?

Yes.

Reason:

- First upload: normal onboarding.
- Rejected re-upload: admin should compare rejection reason.
- Pending replacement: admin should review latest version, not stale upload.
- Verified update: admin should know this does not necessarily block current pickup.

### Should verified customer re-upload immediately invalidate verified status?

Recommended: **No. Keep verified until new upload approved.**

Risk comparison:

| Policy | Operational risk | Security/compliance risk | Recommendation |
|---|---|---|---|
| Immediately invalidate verified on re-upload | High: pickup/contract may be blocked unexpectedly | Lower if strict recertification required | Not recommended for LensBank MVP |
| Keep verified until replacement approved | Lower: avoids unnecessary blocked orders | Medium: staff must know update is pending | Recommended |
| Allow admin/manual override | Lower if well-audited | Depends on policy | Useful later |

Lowest-risk operational default:

- Existing verified state remains usable.
- New upload is a pending update queue item.
- If admin rejects the update, keep old verified document unless admin explicitly invalidates it.

## F. Implementation Readiness

Current Ver1141 is still dry-run/UI-first.

Known limitation:

- Real upload path is not implemented in the customer page.
- Existing upload-session/upload-complete service methods are present, but customer upload flow is not wired to a durable backend versioning policy.

Therefore:

- Readdy may polish UI and copy in preview.
- Readdy must not claim real re-upload/update is live.
- Readdy must not modify DB schema.
- Readdy must not enable production real upload.

## G. Recommendation Summary

1. Support verified re-upload, but as "update pending review".
2. Preserve current verified status until replacement is approved.
3. Use existing `pending` for MVP if no DB migration is allowed.
4. Add upload intent/versioning later before real production upload.
5. Readdy Ver1142 can polish UI copy and CTA visibility in dry-run only.
6. Production remains blocked until backend versioning/upload semantics are implemented and reviewed.
