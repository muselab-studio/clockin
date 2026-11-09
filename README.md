# 員工打卡系統

一個檔案的網頁打卡系統（`index.html`），資料存在 Firebase，可以直接放 GitHub Pages。

- **前台（員工）**：自己申請帳號、登入，打「上班／開始休息／結束休息／下班」，看今天紀錄和本月工時
- **後台（管理員）**：最多 2 個管理帳號；選月份看每個人的出勤天數、總工時、總休息、異常筆數；點進去看每天明細（上班、下班、休息時段、工時）；可補登或刪除打卡；匯出 CSV（Excel 可直接開）

沒填 Firebase 設定時會進入**示範模式**，資料只存在當下那台瀏覽器，適合先試用畫面。

- **排班**：後台「排班表」分頁以月曆排班（可一次排多人、可跨夜），也可以設定休假；同步統計每人每月排班天數、休假天數、排班時數，並和實際工時對照；前台會顯示「今天上班的人員」、今天休假的人，和自己接下來 7 天的班與休假；前台還有整個月的班表（可切換月份，點任一天看那天誰上班、幾點到幾點）

## 排班

- 管理員在後台切到「排班表」，點月曆上任一天即可新增、修改、刪除班次
- 排班時可以勾「居家上班」，前後台月曆上居家的班前面會有小房子圖示，統計表也會列出居家天數（存在班的 `remote` 欄位）
- 前後台月曆上方可一鍵切換「全部／上班／休假」；一格最多顯示 5 人，休假顯示為「名字 休」，所有人都休假的日子只顯示「全員休假」
- 前後台月曆上方有每個人的開關，點名字可以隱藏／顯示那個人的班（只影響自己的畫面，記在自己的瀏覽器）；前台另有「只看我」
- 排班時間用 24 小時制（時、分兩個下拉選單，分鐘每 5 分一格）
- 已排的班和休假都可以直接拖到別天（電腦用滑鼠拖，手機長按後拖），放開後選「複製」或「移動」；班拖到那個人休假的日子會被擋下；休假拖到那個人有班的日子，確認後會一併刪除那個班
- 同一個視窗切到「休假」，選人員和假別（休假、例假、休息日、特休、病假、事假、其他）就能設定休假；休假的人當天不能排班，已經有班的人設為休假時會確認後一併刪除那個班
- 排班工時 = 下班 − 上班 − 休息時間；下班時間早於上班時間時視為跨夜班
- 預設時間 09:30–18:30、休息 60 分鐘，可在「排班預設值」修改；每個班仍可設成任何時間
- 每位人員的顏色可在「人員顏色」自己挑（點色塊），沒設定的人用自動配色；顏色存在 `meta/schedule` 的 `colors`
- 工時提醒（依勞基法與雙週變形工時）：單日排班超過 10 小時、超過 12 小時、連續上班超過 6 天；填了「雙週變形工時週期起始日」後，也會提醒兩週排班超過 80 小時
- 資料存在 Firestore 的 `schedules` 集合（一個班一筆：`uid`、`name`、`date`、`start`、`end`、`breakMin`；休假也存在這裡，`kind` 為 `leave`、假別存在 `leaveType`，不需要另外加規則），預設值存在 `meta/schedule`，不會動到 `users` 和 `punches`

### 啟用排班要加的 Firestore 規則

到 **Firestore → 規則**，在 `match /databases/{database}/documents {` 裡面、其他 `match` 旁邊加上這兩段，然後按「發布」：

```
    // 排班：登入的人都能看班表，只有管理員能排班
    match /schedules/{id} {
      allow read: if request.auth != null;
      allow write: if request.auth != null
        && request.auth.uid in get(/databases/$(database)/documents/meta/admins).data.uids;
    }
    // 排班預設值
    match /meta/schedule {
      allow read: if request.auth != null;
      allow write: if request.auth != null
        && request.auth.uid in get(/databases/$(database)/documents/meta/admins).data.uids;
    }
```

沒加這段的話，打卡功能照常運作，只是後台排班會顯示「沒有權限」、前台看不到班表。

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
