# 員工打卡系統

一個檔案的網頁打卡系統（`index.html`），資料存在 Firebase，可以直接放 GitHub Pages。

- **前台（員工）**：自己申請帳號、登入，打「上班／開始休息／結束休息／下班」，看今天紀錄和本月工時
- **後台（管理員）**：最多 2 個管理帳號；選月份看每個人的出勤天數、總工時、總休息、異常筆數；點進去看每天明細（上班、下班、休息時段、工時）；可補登或刪除打卡；匯出 CSV（Excel 可直接開）

沒填 Firebase 設定時會進入**示範模式**，資料只存在當下那台瀏覽器，適合先試用畫面。

## 工時怎麼算

- 一段「上班 → 下班」算一個班次，歸在上班那天（跨午夜也算同一班）
- 工時 = 上班到下班，扣掉所有休息時間
- 少打上班或下班會標「異常」，管理員可在明細裡補登正確時間
- 員工打卡一律用 Firebase 伺服器時間，改手機時間沒有用

## 上線步驟（約 10 分鐘）

1. 到 [Firebase Console](https://console.firebase.google.com/) 建立專案（可以沿用之前的專案）
2. **Authentication → Sign-in method**：啟用「電子郵件/密碼」
   - 員工不需要真的 email，系統會把帳號轉成 `帳號@clockin.timeflies.local`
3. **Firestore Database**：建立資料庫（地區選 `asia-east1` 台灣）
4. **Firestore → 規則**：把 `firestore.rules` 全部內容貼上並發布
5. **專案設定 → 一般 → 你的應用程式**：新增網頁應用程式，把 `firebaseConfig` 那段貼到 `index.html` 最上方的 `FIREBASE_CONFIG`
6. 上傳到 GitHub Pages（或任何靜態網站空間）
7. **馬上用「管理員」分頁申請 2 個管理帳號**——名額滿了之後就沒人能再申請，避免被別人搶走
8. 第一次有員工打開頁面時，瀏覽器 Console 可能出現一個建立索引的連結，點一下建立即可（沒建也能用，只是讀取比較慢）

## 之後可能會想改的

- 工作室名稱：`STUDIO_NAME`
- 管理帳號上限：`MAX_ADMINS`（也要同步改 `firestore.rules` 裡的 `2`）
- 多久沒打下班視為忘記打卡：`STALE_HOURS`（預設 18 小時）
- 員工忘記密碼：到 Firebase Console → Authentication 找到 `帳號@clockin.timeflies.local` 刪除，請他重新申請（舊打卡紀錄會跟著舊帳號，需要的話可改用新帳號補登）
