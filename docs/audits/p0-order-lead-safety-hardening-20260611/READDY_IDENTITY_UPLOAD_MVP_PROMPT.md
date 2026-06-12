# Readdy Prompt - Identity Upload MVP Customer Flow Only

請只移植 LensBank customer identity upload MVP，不要重構整站，不要覆蓋目前 `lensbank.com.tw` production 後台。

## 背景

LensBank production 目前由 Readdy 發布在：

```text
https://lensbank.com.tw
```

Codex/canonical preview 目前有一個 isolated customer route implementation：

```text
/account/identity?orderId=...
```

本次任務不是 production deploy，而是將 customer identity upload MVP 精準移植到 Readdy production codebase / preview branch，供 owner review。

## GitHub Public Bridge 唯一真相

請使用以下 GitHub public links 作為唯一參考，不要使用本機路徑。

Repo:

```text
https://github.com/juliushsu/lensbank_readdy_public_bridge
```

Branch:

```text
main
```

Handoff 文件：

- [P0_G5B7_READDY_HANDOFF_PACKAGE.md](https://github.com/juliushsu/lensbank_readdy_public_bridge/blob/main/docs/audits/p0-order-lead-safety-hardening-20260611/P0_G5B7_READDY_HANDOFF_PACKAGE.md)
- [READDY_IDENTITY_UPLOAD_MVP_PROMPT.md](https://github.com/juliushsu/lensbank_readdy_public_bridge/blob/main/docs/audits/p0-order-lead-safety-hardening-20260611/READDY_IDENTITY_UPLOAD_MVP_PROMPT.md)
- [IDENTITY_UPLOAD_MVP_ACCEPTANCE_CHECKLIST.md](https://github.com/juliushsu/lensbank_readdy_public_bridge/blob/main/docs/audits/p0-order-lead-safety-hardening-20260611/IDENTITY_UPLOAD_MVP_ACCEPTANCE_CHECKLIST.md)
- [README.md](https://github.com/juliushsu/lensbank_readdy_public_bridge/blob/main/docs/audits/p0-order-lead-safety-hardening-20260611/README.md)

Implementation reference file paths:

- `readdy-frontend/src/router/config.tsx`
- `readdy-frontend/src/pages/account/identity/page.tsx`
- `readdy-frontend/src/components/identity/CustomerIdentityNotice.tsx`
- `readdy-frontend/src/modules/identity/featureFlags.ts`
- `readdy-frontend/src/services/identityGatewayService.ts`
- `readdy-frontend/src/pages/account/checkout/page.tsx`
- `readdy-frontend/src/pages/account/checkout/success/page.tsx`
- `readdy-frontend/src/pages/account/orders/page.tsx`
- `readdy-frontend/src/pages/account/orders/detail/page.tsx`

Note: this public bridge intentionally contains docs only. It does not publish source code, env values, tokens, signed URLs, or DB credentials.

## 絕對限制

請務必遵守：

- 不要 production deploy。
- 不要修改 admin sidebar。
- 不要修改 admin shell。
- 不要修改 admin dashboard/layout/navigation。
- 不要修改 LIFF。
- 不要修改 LINE binding。
- 不要修改 attendance。
- 不要修改 frontend images management。
- 不要修改 storefront sort。
- 不要修改 order status schema。
- 不要新增 DB table。
- 不要修改 DB table。
- 不要新增 migration。
- 不要修改 Supabase RLS。
- 不要開啟 production upload mutation。
- 不要處理真實證件。
- 不要輸出 signed URL、token、cookie、Authorization header。
- 不要把 Codex/pages.dev admin shell 覆蓋到 lensbank.com.tw。

## 只允許做的範圍

請只做以下 customer flow：

1. 新增 authenticated customer route：

```text
/account/identity
/account/identity?orderId=...
```

2. Checkout success 新增 CTA：

```text
完成取件驗證
```

目標：

```text
/account/identity?orderId={orderId}
```

3. Account orders 缺件/待補件 notice CTA 導向：

```text
/account/identity?orderId={orderId}
```

4. Account order detail 缺件/待補件 notice CTA 導向：

```text
/account/identity?orderId={orderId}
```

5. 接入 identity gateway service contract：

```text
GET  /identity/me/status
POST /identity/upload-session
POST /identity/upload-complete
```

6. 加上 feature flag：

```text
VITE_ENABLE_IDENTITY_UPLOAD_MVP
VITE_IDENTITY_UPLOAD_MVP_DRY_RUN
```

7. 預設必須是：

```text
VITE_IDENTITY_UPLOAD_MVP_DRY_RUN=true
```

dry-run true 時：

- 不呼叫 `/identity/upload-session`
- 不呼叫 `/identity/upload-complete`
- 不上傳檔案
- 不呼叫 storage upload
- 只顯示本地 dry-run success state：`已送出審核`

## UX / Copy

請使用以下文案。

### 主標題

```text
完成取件驗證
```

### 信任文案

```text
為了讓您取件更快速、更安心，LensBank 採用數位身份驗證方式。
```

### 不扣留證件文案

```text
完成驗證後，取件時僅需攜帶本人證件到店核對，無須將身分證件留置於店面保管。
```

### 資料用途

```text
您提供的資料僅用於身份驗證、電子契約簽署與租借紀錄管理。系統將依權限進行存取管理。
```

### 上傳欄位

```text
身分證正面
請上傳清楚、完整、未裁切的正面照片。

身分證反面
請上傳清楚、完整、未裁切的反面照片。
```

### 檔案提示

```text
支援 JPG、PNG、WebP，單一檔案上限 8MB。請確認照片清晰且四角完整，避免影響門市審核。
```

### CTA

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

### 成功狀態

```text
已送出審核
我們已收到您上傳的身份資料，門市會接續確認。取件時請攜帶本人證件到店核對。
```

### 狀態文案

Missing:

```text
完成取件驗證
請上傳身分證正反面，讓門市可以完成租借審核與取件安排。
```

Pending:

```text
身份資料已送出，等待門市確認
門市正在確認您的身份資料。若取件日期接近，請留意門市後續通知。
```

Verified:

```text
身份資料已完成驗證
您的身份資料已完成驗證。取件時請攜帶本人證件到店核對。
```

Rejected:

```text
身份資料需重新補件
請重新上傳清楚、完整、未裁切的身分證正面與反面照片。
```

Fallback:

```text
身份資料狀態暫時無法確認
身份資料狀態暫時無法確認，您仍可稍後再試。若取件日期接近，也可以先聯絡門市確認補件方式。
```

## 避免使用

請不要使用：

- `立即完成身份驗證`
- `一般員工無法查看`
- 未經 CTO 確認的法律承諾
- 恐嚇式或懲罰式文案
- 暗示未上傳會立即取消訂單的文案

## UI 行為

`/account/identity?orderId=...` 必須：

- 需要登入。
- 支援 `orderId` query。
- 顯示目前身份狀態。
- 顯示 Missing / Pending / Verified / Rejected / Fallback。
- 顯示信任與資料用途文案。
- 顯示正面 upload control。
- 顯示反面 upload control。
- 未選正反兩面時 `送出審核` disabled。
- 選正反兩面且檔案格式通過後，`送出審核` enabled。
- dry-run true 時，submit 後只顯示 `已送出審核`，不得呼叫任何 upload mutation。
- 可以 `返回訂單`。

檔案限制：

- JPG
- PNG
- WebP
- 單檔 8MB

## Gateway / Service Contract

請使用現有 identity gateway base URL env：

```text
VITE_IDENTITY_GATEWAY_BASE_URL
```

Status:

```text
GET /identity/me/status
```

Upload session:

```text
POST /identity/upload-session
```

Upload complete:

```text
POST /identity/upload-complete
```

重要：

- dry-run true 時不得呼叫 upload-session。
- dry-run true 時不得呼叫 upload-complete。
- dry-run true 時不得 upload file。
- 不得 log signed URL。
- 不得在畫面顯示 signed URL。
- 不得在 console 顯示 token/header/cookie。

## Fallback

如果 `GET /identity/me/status` 失敗：

- 不白屏。
- 不阻斷 orders page。
- 不阻斷 order detail page。
- 顯示：

```text
身份資料狀態暫時無法確認，您仍可稍後再試
```

## Readdy 完成後必須回報

請回報：

1. 修改檔案清單。
2. 是否新增 `/account/identity` route。
3. 是否加入 `VITE_ENABLE_IDENTITY_UPLOAD_MVP`。
4. 是否加入 `VITE_IDENTITY_UPLOAD_MVP_DRY_RUN`。
5. dry-run 預設值是否為 true。
6. Checkout success CTA 是否完成。
7. Account orders CTA 是否完成。
8. Order detail CTA 是否完成。
9. `npm run build` 是否通過。
10. 是否確認 dry-run true 不會呼叫:
    - `/identity/upload-session`
    - `/identity/upload-complete`
    - storage upload
11. 是否確認 admin sidebar 未變。
12. 是否確認 admin shell 未變。
13. 是否確認 LIFF 未變。
14. 是否確認 LINE binding 未變。
15. 是否確認 attendance 未變。

## Owner Acceptance Checklist

完成後 owner 會測：

1. Checkout success 是否有 `完成取件驗證`。
2. 點擊是否到 `/account/identity?orderId=...`。
3. `/account/identity` 是否要求登入。
4. 文案是否正確。
5. 正反面 upload UI 是否出現。
6. 未選兩面是否不能送出。
7. 選兩面後是否可 dry-run submit。
8. dry-run submit 是否顯示 `已送出審核`。
9. dry-run 是否沒有 `/identity/upload-session`。
10. dry-run 是否沒有 `/identity/upload-complete`。
11. dry-run 是否沒有 storage upload。
12. Account orders CTA 是否出現。
13. Order detail CTA 是否出現。
14. Admin sidebar 是否未變。
15. LIFF / attendance 是否未變。
16. LINE binding 是否未變。

## Final Reminder

這次只移植 customer identity upload MVP。

請不要重構整站。
請不要改 admin shell/sidebar。
請不要 production deploy。
請不要開 production upload mutation。
