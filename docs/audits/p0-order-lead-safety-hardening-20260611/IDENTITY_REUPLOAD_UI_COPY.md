# Identity Re-upload UI Copy

Date: 2026-06-12

Purpose:

- Define customer-facing UI copy for missing, fallback, pending, verified, rejected, and dry-run submitted identity states.
- Support future re-upload/update UX without claiming real backend update upload is live.

Scope:

- Copy/design only.
- No DB changes.
- No real upload.
- No production change.

## Copy Principles

1. Avoid fear-based language.
2. Do not imply LensBank keeps physical ID cards.
3. Make "review required" clear after any new upload.
4. For verified update, reassure the customer that existing verification remains valid until the update is reviewed.
5. Do not claim real upload/update is complete in dry-run preview.

## Shared Trust Copy

Primary trust copy:

```text
為了讓您取件更快速、更安心，LensBank 採用數位身份驗證方式。
```

No physical retention copy:

```text
完成驗證後，取件時僅需攜帶本人證件到店核對，無須將身分證件留置於店面保管。
```

Data use copy:

```text
您提供的資料僅用於身份驗證、電子契約簽署與租借紀錄管理。系統將依權限進行存取管理。
```

## State Copy Matrix

### Missing

Title:

```text
完成取件驗證
```

Body:

```text
請上傳身分證正面與反面，讓門市可以完成租借審核與取件安排。
```

Primary CTA:

```text
立即完成取件驗證
```

Alternate CTA:

```text
上傳證件資料
```

Submit button:

```text
送出審核
```

## Fallback

Title:

```text
身份資料狀態暫時無法確認
```

Body:

```text
目前暫時無法確認您的身份資料狀態。您仍可先上傳證件資料；若送出後仍無法確認，請聯繫門市協助。
```

Primary CTA:

```text
先上傳證件資料
```

Submit success title:

```text
已送出資料
```

Submit success body:

```text
我們已收到您的送出動作。若狀態仍無法確認，請聯繫門市協助確認補件狀態。
```

## Pending

Title:

```text
已送出，等待門市確認
```

Body:

```text
門市正在確認您的身份資料。若照片不清楚或需要更換，可重新上傳，將以最新送出的版本為準。
```

Secondary CTA:

```text
重新上傳
```

Replacement submit success:

```text
已重新送出審核
```

Replacement note:

```text
門市將以您最新送出的證件資料進行確認。
```

## Verified

Title:

```text
已完成取件驗證
```

Body:

```text
您的身份資料已完成驗證。取件時請攜帶本人證件到店核對。
```

Secondary CTA:

```text
更新證件資料
```

Update explanation:

```text
若證件換發、照片需更新或門市要求補正，您可以重新上傳證件資料。送出後需由門市重新確認；在新資料通過前，原本已完成的驗證狀態會保留。
```

Update submit success title:

```text
更新資料已送出
```

Update submit success body:

```text
門市會確認您最新送出的證件資料。新資料通過前，原本已完成的驗證狀態會保留。
```

## Rejected

Title:

```text
證件資料需補正
```

Body:

```text
請重新上傳清楚、完整、未裁切的身分證正面與反面照片。
```

Primary CTA:

```text
重新上傳證件
```

Submit success title:

```text
已重新送出審核
```

Submit success body:

```text
我們已收到您重新上傳的身份資料，門市會接續確認。
```

## Submitted / Dry-run Preview

Title:

```text
已送出審核
```

Body:

```text
我們已收到您上傳的身份資料，門市會接續確認。取件時請攜帶本人證件到店核對。
```

Dry-run visible note:

```text
測試模式：本次送出不會上傳真實檔案。
```

Production warning:

```text
此文案只能用於 dry-run preview 或 real upload 已完成的環境；不得在 real upload 尚未實作時宣稱資料已實際上傳。
```

## Admin-facing Labels

Recommended labels for admin review queue:

| Upload intent | Label |
|---|---|
| first upload | 首次上傳 |
| rejected re-upload | 補正重傳 |
| pending replacement | 重新上傳，取代待審版本 |
| verified update | 已驗證客戶更新資料 |
| staff assist | 門市協助上傳 |

## Button Priority

| State | Primary button | Secondary button |
|---|---|---|
| missing | 立即完成取件驗證 | 返回訂單 |
| fallback | 先上傳證件資料 | 聯繫門市 / 返回訂單 |
| pending | 返回訂單 | 重新上傳 |
| verified | 返回訂單 | 更新證件資料 |
| rejected | 重新上傳證件 | 返回訂單 |
| submitted | 返回訂單 | none |

## Terms To Avoid

Avoid:

- `立即完成身份驗證` if it implies instant approval.
- `一般員工無法查看` unless access-control policy is formally approved.
- `保證安全` or broad legal/security guarantees.
- Any copy implying production real upload is live while dry-run is still enabled.
