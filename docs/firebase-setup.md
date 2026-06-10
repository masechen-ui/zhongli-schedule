# Firebase 設定步驟

## 第一步：建立 Firebase 專案

1. 前往 https://console.firebase.google.com/
2. 點「新增專案」
3. 專案名稱輸入：`zhongli-schedule`
4. Google Analytics 可以關閉（不需要）
5. 點「建立專案」

---

## 第二步：建立 Firestore 資料庫

1. 左側選單 → Build → **Firestore Database**
2. 點「建立資料庫」
3. 選「**以正式模式啟動**」
4. 地區選：`asia-east1`（台灣最近）
5. 點「啟用」

---

## 第三步：設定安全性規則

Firestore → 規則 → 貼上以下內容後儲存：

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // 排班資料：登入才能讀寫
    match /schedules/{document=**} {
      allow read, write: if request.auth != null;
    }
    match /metadata/{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

> 目前系統使用自訂登入（工號），未來可以改接 Firebase Auth。
> 暫時開放規則（開發測試用）：
> `allow read, write: if true;`

---

## 第四步：取得設定值

1. 專案設定（齒輪圖示）→「一般」
2. 往下滾到「您的應用程式」→ 點「新增應用程式」→ 選 Web（</>）
3. 應用程式暱稱：`schedule-web`
4. **不要**勾選 Firebase Hosting（我們用 GitHub Pages）
5. 點「註冊應用程式」
6. 複製 `firebaseConfig` 的內容：

```javascript
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "zhongli-schedule.firebaseapp.com",
  projectId: "zhongli-schedule",
  storageBucket: "zhongli-schedule.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

---

## 第五步：填入 index.html

打開 `index.html`，搜尋 `FIREBASE_CONFIG_PLACEHOLDER`，填入上面的設定值。

---

## Firestore 資料結構說明

```
Collection: schedules
  Document: 2026-07          ← 年月 key
    Fields:
      staff:          []     ← 人員陣列
      stores:         []     ← 店家陣列
      groups:         []     ← 組別陣列
      shifts:         []     ← 班別陣列
      schedules:      {}     ← 班表資料
      specialEvents:  []     ← 特殊事件
      staffAssignments: {}   ← 人員指派
      storeViewNotes: {}     ← 店家視角備註
      storeShiftTimes: {}    ← 店家 APOC 時間
      savedAt:        string ← 最後儲存時間
      savedBy:        string ← 誰儲存的

Collection: metadata
  Document: sync
    Fields:
      lastUpdated:    timestamp
      updatedBy:      string   ← empNo
```

---

## 注意事項

- Firestore 免費方案：每日讀取 50,000 次、寫入 20,000 次，排班系統完全夠用
- 資料大小限制：單一文件 1MB，整個月班表遠低於這個限制
- 建議每月建一個 document，不要把所有月份塞在同一個 document
