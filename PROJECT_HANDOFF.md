# 中壢課 自動排班系統 — 專案交接文件
> 建立日期：2026-06-10　最新版本：v8（排班邏輯六項修正 + 人工班表模式對齊）

---

## 🗂 專案識別

| 項目 | 內容 |
|------|------|
| 專案名稱 | 中壢課 自動排班系統 |
| GitHub Repo | `masechen-ui/zhongli-schedule` |
| 線上網址 | `https://masechen-ui.github.io/zhongli-schedule/` |
| Firebase 專案 ID | `schedule-af163` |
| Firebase config | 已填入 index.html（`FIREBASE_CONFIG` 常數，第 174 行） |
| 主程式檔案 | `index.html`（單一檔案，約 4290 行） |

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
                 → Firebase（2秒節流後同步）
                 
開啟 → loadFromStorage() → 本機還原
     → loadFromCloud()   → 雲端載入（覆蓋本機）
     → onSnapshot()      → 即時監聽他人更新
```

---

## 📁 專案結構

```
zhongli-schedule/
├── index.html                    ← 主程式（全部功能在這一個檔案）
├── .github/
│   └── workflows/
│       └── deploy.yml            ← GitHub Actions 自動部署
├── docs/
│   ├── firebase-setup.md         ← Firebase 建立步驟
│   └── deployment-guide.md       ← GitHub 上傳與更新說明
├── .gitignore
└── README.md
```

---

## 🔧 index.html 內部架構

### 區段一覽（行號，v8 版本）

| 行號 | 區段 |
|------|------|
| 1-166 | CSS 樣式 |
| 167-303 | Firebase SDK（`<script type="module">`，含 FIREBASE_CONFIG） |
| 304-336 | HTML 結構（導覽列、頁面容器） |
| 337-345 | 常數定義（品牌、通路、班別清單） |
| 347-406 | STATE 物件、登入邏輯 |
| 408-538 | initData()（預設人員、店家、班別） |
| 539-721 | 工具函式（日期、工時計算、排班週期） |
| 793-1290 | 自動排班核心邏輯（v8 重寫，含主力/副手模式） |
| 1290~ | 導覽列渲染、頁面路由、各頁面 render 函數 |

### 核心 STATE 物件

```javascript
STATE = {
  currentYear, currentMonth,    // 當前年月
  currentPage,                   // 當前頁面
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

### CellData 格式

```javascript
// 上班
{ type:'work', storeId, shiftId, note, locked }
// 公休
{ type:'off' }
// 特休
{ type:'leave', locked:true }
// 開會
{ type:'meeting', locked:true }
// 教育訓練
{ type:'training', locked:true }
```

### v8 排班核心流程（autoScheduleGroup）

```
1. 預處理
   ├── 識別組內員工（staffList）和店家（storeList）
   ├── 計算雙週週期（getBiweeklyPeriods）
   ├── 計算月公休目標（monthOffTarget = 30/14*4 ≈ 9 天）
   └── 月預算分配（staffMonthBudget）
       ├── 每店指派人數統計（storeAssignCount）
       ├── 個人份額 = 店 staffAlloc 扣單店全職後 / 多店人均分
       └── 月目標天數 = 可工作天 × 份額比例

2. 公休預分配（修正I）
   ├── 估算每人平均班別時數（estimateAvgShiftH）
   │   ├── staffAlloc<2 且有 O 班 → 用 O 班時數
   │   ├── 有 P 班 → (A+P)/2
   │   └── 只有 A 班 → A 班時數
   ├── 公休目標 = max(政策4天等比, 雙週天數-80h可上天數)
   └── 等段算法均勻分布

3. 逐日排班（d = 1 ~ days）
   └── 逐店排班（storeList.forEach）
       ├── 候選池構建 + 月預算配速過濾（修正F）
       ├── 主力/副手判斷（修正J）
       │   ├── isPrimaryMode（1<staffAlloc<2 且有單店主力）
       │   │   ├── 主力可用 → 只排主力 1 人（O 班）
       │   │   └── 主力不可用 → 排副手 1 人代班（O 班）
       │   └── 一般模式 → 單店優先排序 + 需求人數
       ├── 班別選擇
       │   ├── 非晚打烊店 → A 班
       │   ├── 主力模式獨自覆蓋 → O 班
       │   ├── staffAlloc<1 → A/P 交替
       │   ├── A/P 互補（即時讀 ms）
       │   └── 自由選班 → 雙週工時配速（修正H）
       ├── 80h 超標防護（先試 P 班，仍超標跳過）
       ├── 店時數配速控管（修正G，非主力模式）
       └── 孤班轉 O（配額砍人後剩 1 人 → 改 O 班）

4. 連班補救（> 6 天連班 → 插入公休）
5. 寫回 STATE.schedules
```

---

## ✅ 已完成功能

### 核心功能
- [x] 動態組別架構（完全不寫死 g1/g2，新增第N組均正常顯示）
- [x] 人員管理（含工號、狀態、excludeAuto，動態顯示所有組別）
- [x] 店家管理（含 APOC 時間、派員設定，動態顯示所有組別）
- [x] 兩週變形工時排班（80h/雙週，錨點可設定）
- [x] 班別：A/P/O/C/B（時數動態依 APOC 計算，非固定 8h）
- [x] 每日派員人數上限（maxStaff + staffAlloc）
- [x] 鎖定格機制（手動修改後重排不覆蓋）
- [x] 特休/開會/訓練 重排後保留
- [x] 動態時數計算（calcShiftHours，依店家 APOC 時間+關店時間）
- [x] 用餐時間動態計算（在店>8h→1h，否則0.5h）

### 排班邏輯（v8 重寫，autoScheduleGroup，基於 1-6 月人工班表分析）
- [x] **修正F：月預算配速分配**：基底改用「可工作天數」（全月-公休），份額改用「扣除單店全職同事後的剩餘比例」，按月進度配速消耗（`staffMonthBudget` + `canAssignToStore` 配速版）
- [x] **修正G：店月時數配速控管**：追蹤每店累積服勤時數，超配額不加派（防止睡衣 127% 超標），但保證 minDaily 出勤；配額砍人後孤班自動轉 O 班
- [x] **修正H：雙週工時配速選班**：A/P 班選擇改用「雙週工時配速」（落後→A 長班、超前→P 短班），加 80h 硬上限防護（超標先試改 P，仍超標跳過）
- [x] **修正I：動態公休天數**：公休目標依「80h 上限 / 平均班別時數」動態計算（正大/融辰 A 班 10.5h 每雙週需 7 休，睡衣 O 班用全天時數估），均勻分布
- [x] **修正J：主力/副手模式**（從人工班表提取）：staffAlloc 1~2 的店（睡衣/莎露/平鎮大等），單店指派人=主力 O 班全天覆蓋，多店指派人=副手只在主力公休日代班
- [x] **v7 沿用**：單人店允許公休、連班補救、A/P 互補即時寫入 ms

### 視角
- [x] 各組班表頁（精簡為排班作業頁，班表折疊）
- [x] 中壢課總表（唯讀，動態顯示所有組別統計）
- [x] 店家視角（班別格可直接點擊修改、代班/支援/備註/AP班可輸入）
- [x] 人員視角（班別格可直接點擊修改、代班行可輸入）
- [x] 歷月班表

### 統計
- [x] 雙週工時達標警示 badge（✅/⚠️）
- [x] 店家時數達成率（概覽頁）
- [x] 非正常狀態人員醒目顯示
- [x] 服勤時數依 APOC 正確計算（非固定 8h）
- [x] 代班次數自動統計

### 其他
- [x] localStorage 自動持久化
- [x] Firebase Firestore 雲端同步（已填入真實 config，多人共用，雲端優先）
- [x] Excel 匯出（SheetJS，色彩班別）
- [x] PDF 列印（@media print 優化，A4 橫向）
- [x] 臨時店點快速新增
- [x] APOC 時間總覽（系統設定）
- [x] GitHub Actions 自動部署

---

## 🔴 待完成功能（優先順序）

### 高優先
- [ ] **上線測試 v8 排班結果**
  - 推上 GitHub 後用真實資料跑 2026/07 排班
  - 重點對照人工班表模式：睡衣主力 O 班、巴斯副手 P 班、融辰/正大純 A 班
  - 雙週法規驗證：連續不超過 6 天、雙週不超過 80h、雙週至少 4 天休息
  - 秋玉組驗證：莎露/平鎮大主力模式是否正確生效

- [ ] **大江華歌爾 O 班模式**
  - 人工班表顯示大江（staffAlloc=3）有 2 人做 A/P 輪替、1 人做 O 班全天
  - 目前自動排班 3 人全做 A/P 輪替，可考慮加入「staffAlloc>=2 的多人店也可指定 1 人 O 班」邏輯

- [ ] **班表格直接點擊修改（人員視角）**
  - 儲存後保留當前選取人員的狀態（returnPage='staff-view' 選人狀態遺失問題）

### 中優先
- [ ] **彈性工時上限設定**
  - 人工排班允許月工時 170-195h（超過 80h/雙週），系統嚴守法定 80h
  - 可考慮加管理員設定：「雙週上限」可調（如 84h/88h），讓排班天數更接近人工排班
  - 這是融辰/正大天數偏低（14-15 天 vs 人工 20 天）的根因

- [ ] **跨月連續排班支援**
  - 目前每月獨立，無法連續計算上月尾的連假天數

- [ ] **代班統計匯出**
  - 依附件規則計算代班次數和支援時數，加入 Excel 匯出

- [ ] **店家 APOC 時間批次編輯**
  - APOC 總覽頁目前唯讀，需要可直接在總覽頁編輯

### 低優先
- [ ] **登入驗證**
  - 目前跳過登入（直接 admin），正式版需要工號驗證
  - `LOGIN_ACCOUNTS` 物件已有，`doLogin()` 已有，只要把跳過的那行移除

- [ ] **特賣會排班支援**
  - 臨時店點已可建立，但排班優先級需要處理

---

## 📊 人工班表分析摘要（v8 依據）

v8 排班邏輯基於 2026 年 1-6 月人工班表的解析結果。以下是關鍵發現：

### 店型排班模式

| 店型 | 派員 | 人工排班模式 | 代表店 |
|------|------|-------------|--------|
| 單人專櫃 | 1.0 | 純 A 班，上 5 休 2 節奏 | 融辰、正大 |
| 半人共用 | 0.5 | A/P 交替，月排約 10 天 | 巴斯、千姿 |
| 主力+副手 | 1.5 | **主力（單店人）O 班全天，副手（多店人）只在主力休時代班** | 睡衣、莎露、平鎮大 |
| 多人百貨 | 2-3 | A/P 輪替為主，部分人 O 班全天覆蓋 | 大江華歌爾、壢 SOGO |

### 關鍵人員排班實例（6 月）

| 人員 | 店 | 班別分布 | 月工時 |
|------|-----|---------|--------|
| 陳淑蓮 | 睡衣（主力） | O 班 16 天、A 2、P 1 | 195.5h |
| 麥菁云 | 巴斯 P 班 10 天 + 睡衣代班 O 班 9 天 | 巴斯+睡衣合計 | 191h |
| 楊舒菁 | 莎露（主力） | O 班 18 天 | ~190h |
| 李沛儀 | 平鎮大（主力） | O 班 20 天 | ~195h |
| 吳宜君 | 融辰 | A 班 20 天 | 170h |
| 孫心蘋 | 正大 | A 班 20 天 | 170h |

### 自動排班 vs 人工排班差距

人工排班月工時 170-195h（超過雙週 80h 法定上限），自動排班嚴守 80h，因此：
- 融辰/正大：自動 14-15 天 vs 人工 20 天（差距 5 天）
- 睡衣主力：自動 16 天 vs 人工 19 天（差距 3 天）
- 如需縮小差距，可考慮加入「彈性工時上限」設定

---

## 🚀 部署步驟（首次）

1. **Firebase**：已建立，專案 ID `schedule-af163`
2. **GitHub**：Repo `masechen-ui/zhongli-schedule` 已建立
3. **GitHub Pages**：已啟用，網址 `https://masechen-ui.github.io/zhongli-schedule/`
4. **日常更新**：修改 index.html → git push → 30 秒後自動部署

## 🔄 日常更新

```bash
git add index.html
git commit -m "說明更新了什麼"
git push
# → 30 秒後網站自動更新
```

---

## 🐛 已知問題記錄

| 版本 | 問題 | 狀態 |
|------|------|------|
| v8 | 睡衣搶光麥菁云全部天數，巴斯 0 天排不到人（月預算用全月 30 天當基底，預算永遠用不完） | ✅ v8 修復（修正F：可工作天基底 + 份額重算 + 按月進度配速） |
| v8 | 睡衣店時數 127% 超標（dailyMax=2 每天都派 2 人，實際變派員 2.0） | ✅ v8 修復（修正G：店月時數配速控管 + 孤班轉 O 班） |
| v8 | 大江人員月工時僅 131-147h，公休超過（連吃 A 班 10h，8 天撞 80h 上限，後段整片公休） | ✅ v8 修復（修正H：選班改用雙週工時配速 + 80h 硬上限防護） |
| v8 | 正大/融辰公休堆在雙週尾端連三休（公休固定 4 天/雙週，但 A 班 10.5h 只夠上 7 天） | ✅ v8 修復（修正I：公休天數依 80h/平均時數 動態計算） |
| v8 | 睡衣/莎露/平鎮大等 1.5 派員店：系統平均分配兩人，不符人工班表「主力 O 班 + 副手代班」模式 | ✅ v8 修復（修正J：主力/副手模式，從 1-6 月人工班表提取邏輯） |
| v7 | 多店指派員工（麥菁云：睡衣+巴斯）巴斯排不到人 | ⚠️ v7 部分修復 → v8 徹底修復 |
| v7 | 正大/融辰單人店 canTakeOff 永遠 false → 前段連班無公休 | ✅ v7 修復（staffAlloc<=1 允許公休） |
| v7 | 公休集中前段或後段，未均勻分布 | ⚠️ v7 部分修復 → v8 徹底修復（動態公休天數） |
| v7 | 連班補救 else if 條件永遠不成立（邏輯 bug） | ✅ v7 修復 |
| v7 | A/P互補：toAssign 前面的人未寫入 ms，互補失效 | ✅ v7 修復（立即寫入ms） |
| v6 | 人員/店家/指派頁面只顯示 g1/g2，新增組別不出現 | ✅ v6 修復（動態迴圈所有組別） |
| v6 | 中壢課總表工時統計寫死貞文/秋玉組名 | ✅ v6 修復（動態產生） |
| v6 | 特殊安排 modal 人員下拉顯示錯誤組名 | ✅ v6 修復 |
| v6 | Firebase FIREBASE_CONFIG 變數名稱不一致 → 雲端同步失效 | ✅ v6 修復 |
| v5 | STATE.shifts 未存入 localStorage → 班別全顯示「?」 | ✅ v5 修復 |
| v5 | 特休日期解析時區偏移 → 特休消失 | ✅ v5 修復 |
| v4 | 重排後特休消失 | ✅ v4 修復 |
| v3 | saveCellEdit 後跳回組班表頁 | ✅ v3 修復 |

---

## 📞 相關帳號資訊

- GitHub 帳號：masechen-ui
- Firebase 專案 ID：schedule-af163
- 線上網址：https://masechen-ui.github.io/zhongli-schedule/
