@AGENTS.md

# LIFF 作品集專案

LINE LIFF 登入的作品集網站：瀏覽統計、收藏、投票（每人最多三票）。
技術：Next.js 16 App Router + TypeScript + Tailwind + Supabase + LINE LIFF，部署在 Vercel。
學習手冊：`../LIFF 作品集專案複習手冊.md`（14 天計畫、架構、SQL、面試問題）。

## 學習模式（最重要）

- 我是在學習中的開發者。每一步先用繁體中文解釋「為什麼」，再動手。
- 核心邏輯留給我寫：`verifyLineIdToken`、`record_view`、`cast_vote`、`toggleFavorite` 的主體。
  你給提示、骨架、TODO 註解，我寫完你再 review。UI 與樣板可以直接產生。
- 每個功能先進 plan mode，列出要動哪些檔案，我看懂才同意。
- 功能寫完出 3 題小測驗。

## 安全規則（不可違反）

- 所有寫入資料的 Server Action，第一步必須是 `verifyLineIdToken(idToken)`；
  絕對不信任前端傳來的 userId 或 `liff.getProfile()` 的資料。
- Supabase secret 金鑰只能出現在有 `import 'server-only'` 的檔案。
- 只有 LIFF ID 能用 `NEXT_PUBLIC_` 前綴；Supabase 秘密金鑰、LINE Channel ID 不行。
- 所有表都開 RLS 且不寫公開 policy；資料庫函式要 `revoke execute ... from anon, authenticated`。
- E2E 測試用的固定測試 Token 只能在測試環境啟用，正式環境必須關閉。

## 程式風格

- 不用 `any`。型別不確定時說明應該是什麼型別、為什麼。
- 讀資料用 Server Component，寫資料用 Server Action，只有需要瀏覽器（LIFF、點擊、state）才用 `'use client'`，且 Client Component 盡量小。
- Next.js 16：`params` 是 Promise 要 await；`middleware.ts` 已改名 `proxy.ts`。
- SQL 一律寫成 `supabase/migrations/` 下的 migration 檔，不只在 SQL Editor 貼。
- 每完成一天的目標 commit 一次，commit 訊息可用中文。
