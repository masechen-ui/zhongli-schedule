# 中壢課排班系統 — 版本歷史與 Bug 記錄

---

## v9（2026-06-12）Firebase 雲端同步修復

| 問題 | 修法 |
|------|------|
| `const STATE` 無法被 Firebase `<script type="module">` 存取，`window.STATE` 永遠是 undefined | 改為 `window.STATE =`（第 338 行） |
| onSnapshot → loadFromCloud → autoSave → saveToCloud 無限循環 | 加入 `_cloudLoadingLock` 旗標，載入期間阻止 autoSave 回寫；loadFromCloud 完成後只寫 localStorage |
| Race condition：`_lastSavedAt` 在 `await setDoc` 之後才設定，onSnapshot 先收到通知無法識別為自己推的 | `_lastSavedAt` 移到 `setDoc` 之前設定；onSnapshot 比對改用 2 秒容錯視窗 |

---

## v8（2026-06-10）排班邏輯六項修正

| 問題 | 修法 |
|------|------|
| 睡衣搶光麥菁云全部天數，巴斯 0 天（月預算用全月 30 天當基底） | 修正F：可工作天基底 + 份額重算 + 按月進度配速 |
| 睡衣店時數 127% 超標（dailyMax=2 每天都派 2 人） | 修正G：店月時數配速控管 + 孤班轉 O 班 |
| 大江人員月工時僅 131-147h（A 班 10h，8 天撞 80h 上限，後段整片公休） | 修正H：選班改用雙週工時配速 + 80h 硬上限防護 |
| 正大/融辰公休堆在雙週尾端連三休（A 班 10.5h 只夠上 7 天） | 修正I：公休天數依 80h/平均時數 動態計算 |
| 睡衣/莎露/平鎮大 1.5 派員店：系統平均分配，不符「主力O班+副手代班」 | 修正J：主力/副手模式 |
| 多店指派員工（麥菁云）巴斯排不到人 | v7 部分修復 → v8 徹底修復 |

---

## v7

| 問題 | 修法 |
|------|------|
| 正大/融辰單人店 canTakeOff 永遠 false → 前段連班無公休 | staffAlloc<=1 允許公休 |
| 公休集中前段或後段 | 部分修復（v8 徹底修復） |
| 連班補救 else if 條件永遠不成立 | 邏輯修正 |
| A/P 互補：toAssign 前面的人未寫入 ms，互補失效 | 立即寫入 ms |

---

## v6

| 問題 | 修法 |
|------|------|
| 人員/店家/指派頁面只顯示 g1/g2，新增組別不出現 | 動態迴圈所有組別 |
| 中壢課總表工時統計寫死貞文/秋玉組名 | 動態產生 |
| 特殊安排 modal 人員下拉顯示錯誤組名 | 修正 |
| Firebase FIREBASE_CONFIG 變數名稱不一致 → 雲端同步失效 | 統一變數名稱 |

---

## v5

| 問題 | 修法 |
|------|------|
| STATE.shifts 未存入 localStorage → 班別全顯示「?」 | 補存 shifts |
| 特休日期解析時區偏移 → 特休消失 | 修正時區處理 |

---

## v4

| 問題 | 修法 |
|------|------|
| 重排後特休消失 | 修復 |

---

## v3

| 問題 | 修法 |
|------|------|
| saveCellEdit 後跳回組班表頁 | 修復 returnPage 邏輯 |
