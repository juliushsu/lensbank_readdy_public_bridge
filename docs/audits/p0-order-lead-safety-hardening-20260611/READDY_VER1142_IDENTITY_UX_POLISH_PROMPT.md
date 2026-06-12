# Readdy Prompt - Ver1142 Identity Re-upload UX Polish

請只做 customer identity upload/re-upload UX polish，不要部署 production，不要改 DB，不要改 admin shell/sidebar，不要開 real upload。

## 背景

Ver1141 已讓：

- `/account/profile` 出現補件入口。
- `/account/identity` 在 fallback 狀態仍可顯示 upload form。

Owner preview 發現下一個 UX 問題：

客戶未來可能需要重新上傳或更新證件，例如：

- 身分證換發
- 重新拍照
- 原資料不清楚
- 門市要求補正
- 客戶主動更新資料

本輪請做 UI/dry-run polish，不要聲稱 real re-upload 已上線。

## 絕對限制

請遵守：

- 不要 production deploy。
- 不要新增或修改 DB schema。
- 不要新增 migration。
- 不要修改 Supabase RLS。
- 不要開啟 production real upload。
- 不要處理真實證件。
- 不要輸出 signed URL、token、cookie、Authorization header、storage path。
- 不要改 admin sidebar。
- 不要改 admin shell。
- 不要改 LIFF。
- 不要改 attendance。
- 不要改 LINE binding。
- 不要改 frontend images。
- 不要改 storefront sort。
- 不要聲稱 real update upload 已上線。

## 參考文件

請閱讀 public bridge repo 中的文件：

- `IDENTITY_REUPLOAD_LOGIC_DESIGN.md`
- `IDENTITY_REUPLOAD_UI_COPY.md`
- `READDY_VER1142_IDENTITY_UX_POLISH_PROMPT.md`

## UI Polish Goal

請在 dry-run preview 中調整 customer-facing identity UI，使不同狀態的 CTA 更符合後續 re-upload/update 邏輯。

### Missing

顯示：

```text
完成取件驗證
```

CTA:

```text
立即完成取件驗證
```

或：

```text
上傳證件資料
```

### Fallback

顯示：

```text
身份資料狀態暫時無法確認
```

CTA:

```text
先上傳證件資料
```

說明：

```text
目前暫時無法確認您的身份資料狀態。您仍可先上傳證件資料；若送出後仍無法確認，請聯繫門市協助。
```

### Pending

顯示：

```text
已送出，等待門市確認
```

CTA:

```text
重新上傳
```

說明：

```text
若照片不清楚或需要更換，可重新上傳，將以最新送出的版本為準。
```

### Verified

顯示：

```text
已完成取件驗證
```

CTA:

```text
更新證件資料
```

說明：

```text
若證件換發、照片需更新或門市要求補正，您可以重新上傳證件資料。送出後需由門市重新確認；在新資料通過前，原本已完成的驗證狀態會保留。
```

### Rejected

顯示：

```text
證件資料需補正
```

CTA:

```text
重新上傳證件
```

說明：

```text
請重新上傳清楚、完整、未裁切的身分證正面與反面照片。
```

## Status Logic Guidance

UI should support these user paths in preview:

| Current state | CTA | Preview behavior |
|---|---|---|
| missing | 上傳證件資料 | Show upload form, dry-run submit to submitted state |
| fallback | 先上傳證件資料 | Show upload form, dry-run submit to submitted/fallback submitted state |
| rejected | 重新上傳證件 | Show upload form, dry-run submit to submitted state |
| pending | 重新上傳 | Show upload form after user confirms replacement intent |
| verified | 更新證件資料 | Show update explanation, then upload form after user confirms |

For pending/verified:

- Do not imply current status is immediately changed in production.
- Explain this is a new review request.
- In preview/dry-run, local submitted state is acceptable.

## Backend Reality Warning

If real backend does not yet support versioning:

- Keep this as UI/dry-run only.
- Do not call upload-session/upload-complete in dry-run.
- Do not claim the document was actually updated.
- Do not change DB schema.
- Do not enable production real upload.

## Recommended Low-risk Copy

Trust copy:

```text
為了讓您取件更快速、更安心，LensBank 採用數位身份驗證方式。
```

No physical retention:

```text
完成驗證後，取件時僅需攜帶本人證件到店核對，無須將身分證件留置於店面保管。
```

Data use:

```text
您提供的資料僅用於身份驗證、電子契約簽署與租借紀錄管理。系統將依權限進行存取管理。
```

Dry-run visible note:

```text
測試模式：本次送出不會上傳真實檔案。
```

## Acceptance Checklist

Readdy 回報時請列出：

1. 修改檔案清單。
2. `missing` 狀態畫面與 CTA。
3. `fallback` 狀態畫面與 CTA。
4. `pending` 狀態是否可進入重新上傳 UI。
5. `verified` 狀態是否可看到更新證件資料 CTA。
6. `rejected` 狀態是否可重新上傳。
7. dry-run submit 是否仍不呼叫 upload-session/upload-complete/storage upload。
8. 是否沒有改 admin shell/sidebar。
9. 是否沒有改 LIFF、attendance、LINE binding、frontend images、storefront sort。
10. build 結果。

## Production Gate

Production 前仍需：

- backend versioning / replacement policy。
- admin review queue 能區分 first upload / re-upload / update。
- audit log。
- upload-session/upload-complete/storage upload 真實流程 staging test。
- owner approval to disable dry-run。
