# 🚀 部署與更新操作說明

## 一次性設定（只做一次）

### Step 1：建立 GitHub Repo

1. 前往 https://github.com → 右上角「+」→「New repository」
2. Repository name：`zhongli-schedule`
3. 設為 **Public**（GitHub Pages 免費版需要 Public）
4. 不要勾選 Initialize，直接「Create repository」

### Step 2：上傳檔案

把整個 `zhongli-schedule` 資料夾的內容上傳：

**方法 A：網頁拖拉（最簡單）**
1. 打開剛建立的 repo 頁面
2. 把資料夾裡的所有檔案拖進瀏覽器視窗
3. 最下面填 Commit message：`初始版本`
4. 點「Commit changes」

**方法 B：用 Git（推薦，之後更新更方便）**
```bash
cd zhongli-schedule
git init
git add .
git commit -m "初始版本"
git branch -M main
git remote add origin https://github.com/你的帳號/zhongli-schedule.git
git push -u origin main
```

### Step 3：開啟 GitHub Pages

1. Repo 頁面 → 上方「Settings」
2. 左側「Pages」
3. Source 選：**GitHub Actions**
4. 儲存

→ 第一次 push 後，約 **1 分鐘** 就可以用網址開啟

### Step 4：確認網址

- 格式：`https://你的帳號.github.io/zhongli-schedule/`
- 可以在 Settings → Pages 看到

---

## 日常更新流程

**每次修改完 index.html 後：**

### 方法 A：GitHub 網頁直接編輯（最快）

1. 打開 GitHub repo
2. 點 `index.html`
3. 右上角鉛筆圖示「Edit this file」
4. 貼上新版本內容（全選 → 貼上）
5. 點「Commit changes」
6. → **30秒後** 自動部署完成

### 方法 B：重新上傳檔案

1. 打開 GitHub repo
2. 點 `index.html` → 右上角垃圾桶刪除
3. 把新的 `index.html` 拖進去上傳
4. Commit → 自動部署

### 方法 C：Git 指令（最標準）

```bash
# 修改完後
git add index.html
git commit -m "v5.x 說明本次更新了什麼"
git push
```

---

## Firebase 填入步驟

完成 Firebase 設定（參見 docs/firebase-setup.md）後：

1. 打開 `index.html`
2. 搜尋 `YOUR_API_KEY`
3. 把整個 `FIREBASE_CONFIG` 物件替換為你的設定值
4. 儲存 → Commit → 推上去

---

## 確認部署狀態

1. Repo 頁面 → 上方「Actions」分頁
2. 看最新一筆是否有 ✅ 綠色勾勾
3. 如果是 ❌ 紅色，點進去看錯誤訊息

---

## 給組長的使用說明

網址建立後，把以下資訊傳給組長：

```
中壢課排班系統網址：
https://[你的帳號].github.io/zhongli-schedule/

使用說明：
- 用 Chrome 或 Edge 開啟
- 可加入書籤或釘選到手機首頁
- 資料會自動儲存（有雲端同步後三人共用同一份資料）
- 建議每個月定期到「系統設定 → 匯出備份 JSON」存一份到電腦
```
