# 中壢課 自動排班系統

華歌爾集團中壢分區 排班管理系統
涵蓋 Wacoal / Savvy / BeenTeen，貞文組（7間）+ 秋玉組（10間）

---

## 🌐 線上網址

> 部署後填入：`https://masechen-ui.github.io/zhongli-schedule/`

---

## 🔥 Firebase 設定

資料同步使用 Firebase Firestore。

**需要填入的設定值**（在 `index.html` 的 `FIREBASE_CONFIG` 區塊）：

```
apiKey: "AIzaSyAouhpSMvx_5hLgcyKMIXby3xnrSlLwXAY",
  authDomain: "schedule-af163.firebaseapp.com",
  projectId: "schedule-af163",
  storageBucket: "schedule-af163.firebasestorage.app",
  messagingSenderId: "998984586614",
  appId: "1:998984586614:web:b57156c73c4719d8529657",
  measurementId: "G-ZBECW54RBK"
```

**Firestore 資料結構：**
```
schedules/
  {2026}-{07}/          ← 例如 2026-07
    staff[]
    schedules{}
    specialEvents[]
    ...

metadata/
  lastUpdated              ← 最後更新時間（用於衝突偵測）
```

---

## 🚀 部署流程

### 首次設定
1. Fork 或 Clone 此 repo
2. GitHub Settings → Pages → Source 選 **GitHub Actions**
3. 推一次 commit，Actions 自動部署

### 日常更新
```bash
# 修改完 index.html 後
git add index.html
git commit -m "v5.x 更新說明"
git push
# → GitHub Actions 自動部署，約 30 秒後生效
```

---

## 👥 使用者

| 身份 | 帳號 | 權限 |
|------|------|------|
| 指導員（Tim）| 03250 | 管理員，可看所有組 |
| 王貞文 | 19310 | 貞文組組長 |
| 廖秋玉 | 12773 | 秋玉組組長 |

---

## 📁 檔案結構

```
zhongli-schedule/
├── index.html              ← 主程式（單一檔案）
├── .github/
│   └── workflows/
│       └── deploy.yml      ← GitHub Actions 自動部署
├── docs/
│   └── firebase-setup.md   ← Firebase 設定說明
└── README.md
```

---

## 🛠 技術架構

- 純前端 HTML/CSS/JavaScript（無框架）
- localStorage 本機備份
- Firebase Firestore 雲端同步（多人共用）
- GitHub Pages 靜態托管
- GitHub Actions 自動 CI/CD
