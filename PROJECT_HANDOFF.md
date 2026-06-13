# 中壢課 自動排班系統 — 專案交接文件
> 建立日期：2026-06-10　最新版本：v13（三視角細節修正）

---

## 🗂 專案識別

| 項目 | 內容 |
|------|------|
| 專案名稱 | 中壢課 自動排班系統 |
| GitHub Repo | `masechen-ui/zhongli-schedule` |
| 線上網址 | `https://masechen-ui.github.io/zhongli-schedule/` |
| Firebase 專案 ID | `schedule-af163` |
| Firebase config | 已填入 index.html（`FIREBASE_CONFIG` 常數，第 174 行） |
| Firestore 規則 | `allow read, write: if true`（已設定，無需 Auth） |
| 主程式檔案 | `index.html`（單一檔案，約 4580 行） |

---

## 🏢 業務背景

- **公司**：華歌爾集團 中壢分區（桃園販賣部）
- **Tim 工號**：03250（管理員）
- **品牌**：Wacoal / Savvy / BeenTeen / 莎露 / 金華歌爾
- **組織**：
  - 貞文組：王貞文 19310（組長）+ 7間店
  - 秋玉組：廖秋玉 12773（組長）+ 10間店
- **共計**：32名員工、17間店（不含辦公室）

---

## 🏗 技術架構

```
純前端單一 HTML 檔案
├── HTML/CSS/JS（無框架）
├── localStorage  → 本機自動備份
├── Firebase Firestore → 雲端同步（多人共用）
├── GitHub Pages  → 靜態托管
└── GitHub Actions → 推上去自動部署
```

### 資料流

```
操作 → autoSave() → localStorage（立即）
                 → Firebase（2秒節流後同步，_cloudLoadingLock 鎖定中則跳過）

開啟 → loadFromStorage() → 本機還原
     → loadFromCloud()   → 雲端載入（覆蓋本機）
                              ↓ 設 _cloudLoadingLock=true
                              ↓ 只寫 localStorage（不觸發 autoSave）
                              ↓ 3秒後解鎖
     → onSnapshot()      → 即時監聽他人更新
                              ↓ 比對 _lastSavedAt 時間差 < 2秒 → 忽略（自己推的）
                              ↓ 時間差 >= 2秒 → loadFromCloud()
```

**防無限循環機制（v9）：**
- `_lastSavedAt`：在 `setDoc` 之前設定，onSnapshot 收到時比對時間差，< 2 秒視為自己推的
- `_cloudLoadingLock`：loadFromCloud 期間鎖定，阻止 autoSave 回寫雲端，3 秒後解鎖

---

## 🔧 index.html 內部架構

### 區段一覽

| 行號 | 區段 |
|------|------|
| 1-185 | CSS 樣式（含 v10 圓角/陰影 token） |
| 167-303 | Firebase SDK（`<script type="module">`，含 FIREBASE_CONFIG） |
| 304-336 | HTML 結構（導覽列、頁面容器） |
| 337-345 | 常數定義（品牌、通路、班別清單） |
| 371~ | 圖示系統（`ICON_PATHS` + `icon()`，v10） |
| 338 | `window.STATE` 宣告（注意：必須是 window.STATE，不能是 const） |
| 347-406 | STATE 物件、登入邏輯 |
| 408-538 | initData()（預設人員、店家、班別） |
| 539-721 | 工具函式（日期、工時計算、排班週期） |
| 793-1290 | 自動排班核心邏輯（v8，含主力/副手模式） |
| 1290~ | 導覽列渲染、頁面路由、各頁面 render 函數 |

### 核心 STATE 物件

```javascript
window.STATE = {
  currentYear, currentMonth,
  currentPage,
  login: { loggedIn, empNo, role, name },
  staff: [ { id, empNo, name, role, groupId, status, excludeAuto } ],
  stores: [ { id, storeNo, name, brand, channel, groupId,
              staffAlloc, maxStaff, minDaily, target,
              shiftTimes, isTemp, tempStart, tempEnd } ],
  shifts: [ { id, name, code, start, end, hours } ],
  groups: [ { id, name, leaderEmpNos, color } ],
  schedules: { "2026-07": { staffId: { day: CellData } } },
  specialEvents: [ { id, staffId, date, type, note } ],
  staffAssignments: { staffId: [storeId, ...] },
  storeViewNotes: { "storeId_staffId_2026-07": { dep_1, sup_1, note_1, ap_1 } },
  storeShiftTimes: {},
  biweeklyAnchor: "2025-07-05",
  biweeklyOverrides: {},
}
```

---

## ✅ 已完成功能

### 核心
- [x] 動態組別架構（完全不寫死 g1/g2）
- [x] 兩週變形工時排班（80h/雙週，錨點可設定）
- [x] 班別 A/P/O/C/B（時數動態依 APOC 計算）
- [x] 鎖定格機制、特休/開會/訓練重排後保留
- [x] Firebase 雲端同步（多人共用，防無限循環，v9）
- [x] localStorage 自動持久化
- [x] Excel 匯出（SheetJS）、PDF 列印
- [x] GitHub Actions 自動部署

### 介面（v10）
- [x] 統一 SVG 圖示系統（取代全站 emoji）
- [x] 圓角尺度統一 + 染色陰影層次
- [x] 按鈕系統收斂 + 按壓回饋
- [x] header／統計卡精修（F）
- [x] 三視角統一呈現（`renderPersonBlock` 共用區塊，總表依組分群，v11）

### 排班邏輯（v8，基於 1-6 月人工班表）
- [x] 月預算配速分配（修正F）
- [x] 店月時數配速控管（修正G）
- [x] 雙週工時配速選班（修正H）
- [x] 動態公休天數（修正I）
- [x] 主力/副手模式（修正J）

---

## 🔴 待完成功能

### 高優先
- [x] **三視角統一呈現格式**（共用 `renderPersonBlock`；總表已改依組分群每人豐富區塊，v11）
- [ ] **上線測試 v8 排班結果**（睡衣主力O班、巴斯副手P班、融辰/正大純A班）
- [ ] **大江華歌爾 O 班模式**（staffAlloc=3，目前全A/P，需1人O班）
- [ ] **人員視角班表格點擊後保留選取狀態**

### 中優先
- [ ] **彈性工時上限設定**（雙週上限可調 84h/88h，縮小與人工排班差距）
- [ ] **跨月連續排班支援**
- [ ] **代班統計匯出**
- [ ] **店家 APOC 時間批次編輯**

### 低優先
- [ ] **favicon.ico 404**（加 `<link rel="icon" href="data:,">` 消除）
- [ ] **頁面載入後推送一次 Firestore**（可改為「雲端有資料就不推送」）
- [ ] **登入驗證**（`LOGIN_ACCOUNTS` 和 `doLogin()` 已有，移除跳過那行即可）
- [ ] **特賣會排班支援**

---

## 🚀 部署

```bash
# 日常更新
git add index.html
git commit -m "說明更新內容"
git push
# → 30 秒後自動部署至 GitHub Pages
```

---

## 📞 帳號資訊

- GitHub：masechen-ui
- Firebase 專案 ID：schedule-af163
- 線上網址：https://masechen-ui.github.io/zhongli-schedule/

---

> 歷史 bug 記錄請見 `CHANGELOG.md`
