@AGENTS.md

# LINE 自助洗衣系統（line-liff-laundry）

在 LINE 裡掃洗衣機 QR Code 啟動機台、即時看機台狀態與倒數、洗好時收到 LINE 推播。
技術：Next.js 16 App Router + TypeScript + Tailwind + Supabase（PostgreSQL、Realtime、pg_cron）+ LINE LIFF + Messaging API，部署在 Vercel。
學習手冊：`../LINE 自助洗衣專案手冊.md`（18 天計畫、架構、資料表、核心函式、測試、面試問題）。
GitHub：https://github.com/michael9487/line-liff-laundry（push 到 `main` 會自動部署）
正式網址：https://line-liff-laundry.vercel.app（LIFF Endpoint URL 用這個）

## 學習模式（最重要）

- 一律用繁體中文。我是正在學習的開發者，每一步先解釋「為什麼」再動手。
- 手冊中標示 ✍️ 的核心邏輯由我自己寫：`MachineCard`、`verifyLineIdToken`、`start_session`、
  `complete_due_sessions`、`isValidSignature`。只給骨架、提示、TODO 註解，我寫完再 review，不要直接給答案。
  UI、樣板、設定檔可以直接產生。
- 每個功能先進 plan mode，列出要動哪些檔案，我看懂才同意。
- 功能寫完出 3 題小測驗。

## 安全規則（不可違反）

- 所有寫入資料的 Server Action，第一步必須是 `verifyLineIdToken(idToken)`；
  絕對不信任前端傳來的 userId 或 `liff.getProfile()` 的資料。
- 瀏覽器不能寫資料庫。RLS 全部開啟，只有 `machines`、`programs` 有公開讀取的 policy，其他表沒有 policy。
- Supabase secret 金鑰只能出現在有 `import 'server-only'` 的檔案。
- 只有 `NEXT_PUBLIC_LIFF_ID`、`NEXT_PUBLIC_SUPABASE_URL`、`NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` 可以用 `NEXT_PUBLIC_` 前綴。
- 資料庫函式都要 `revoke execute ... from public, anon, authenticated`。
- `/api/cron/tick` 必須檢查 `CRON_SECRET`；`/api/line/webhook` 必須用原始 body 驗證 `x-line-signature`（timingSafeEqual）。
- 密鑰不能寫進 migration（會進 Git），pg_cron 用到的網址和密鑰放 Supabase Vault。
- 管理功能要驗證 Token 後再檢查 `ADMIN_LINE_USER_IDS`。
- E2E 測試用的固定測試 Token 只能在測試環境啟用，正式環境必須關閉。

## 資料庫規則

- 「檢查 + 寫入」要放在資料庫函式，用 `select ... for update` 鎖住；鎖的順序固定為「先機台、後使用者」。
- 點數異動一定同時寫 `wallet_transactions`。
- SQL 一律寫成 `supabase/migrations/` 下的 migration 檔，不只在 SQL Editor 貼。

## 程式風格

- 不用 `any`。型別不確定時說明應該是什麼型別、為什麼。
- 讀公開資料用 Server Component；寫資料用 Server Action；只有需要瀏覽器（LIFF、點擊、倒數、Realtime）才用 `'use client'`，且 Client Component 盡量小。
- Next.js 16：`params` 是 Promise 要 await；`middleware.ts` 已改名 `proxy.ts`。寫之前先查 `node_modules/next/dist/docs/`。
- 每完成一天的目標 commit 一次，commit 訊息可用中文。
