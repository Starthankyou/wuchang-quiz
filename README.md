# 即時文字雲問卷系統 — 無常

## 檔案說明

| 檔案 | 用途 |
|---|---|
| `form.html` | 學員掃 QR code 後填答（手機） |
| `display.html` | 投影幕即時文字雲 |
| `worker.js` | Cloudflare Worker，Claude API key 藏在這裡，不進 git |

---

## 為什麼要 Cloudflare Worker？

GitHub Pages 是 public repo → 所有程式碼公開 → API key 不能放裡面。

解法：讓 `display.html` 呼叫你的 Worker，Worker 再去呼叫 Claude API。Key 存在 Cloudflare 的環境變數，永遠不會出現在 GitHub。

```
display.html  →  Cloudflare Worker  →  Claude API
(GitHub 公開)     (key 在環境變數)
```

---

## 完整部署步驟

### Step 1：Firebase Realtime Database

1. https://console.firebase.google.com → 建立專案
2. Realtime Database → 建立 → 測試模式
3. 規則設為公開讀寫：
```json
{ "rules": { ".read": true, ".write": true } }
```
4. 專案設定 → 複製 `firebaseConfig`

---

### Step 2：Cloudflare Worker（保護 Claude API Key）

1. https://dash.cloudflare.com 免費註冊
2. Workers & Pages → Create → Create Worker
3. 把 `worker.js` 全部貼進去 → Deploy
4. 複製 Worker URL（`https://xxx.workers.dev`）
5. Worker → Settings → Variables → Add variable
   - 類型：**Secret**
   - 名稱：`CLAUDE_API_KEY`
   - 值：你的 `sk-ant-...`
   - Save and deploy

---

### Step 3：填入 display.html

```javascript
// 找到這行，換成你的 Worker URL
const WORKER_URL = "https://你的worker網址/merge";
```

---

### Step 4：兩個 HTML 填入 Firebase config

`form.html` 和 `display.html` 裡的 `firebaseConfig` 換成你的。

> Firebase 的 apiKey 放 public repo 是 OK 的，它只是識別專案用。
> 真正要藏的是 Claude API key，已經放在 Cloudflare 了。

---

### Step 5：推上 GitHub Pages

```bash
git init
git add form.html display.html README.md
git commit -m "init"
git remote add origin https://github.com/你的帳號/repo.git
git push -u origin main
```

GitHub → Settings → Pages → Branch: main → Save

- 填答：`https://帳號.github.io/repo/form.html`
- 投影：`https://帳號.github.io/repo/display.html`

---

### Step 6：現場使用

| 誰 | 動作 |
|---|---|
| 你 | 電腦開 display.html 投影 |
| 學員 | 手機掃 QR 碼 → 填 form.html |
| 你 | 右上角切換「無常聯想」/「無常意義」 |
| 系統 | 自動合併同義詞 → 更新文字雲 |

---

### 活動結束

1. Firebase Rules 改回需要驗證
2. Cloudflare Worker 停用
3. Anthropic console 撤銷 API key
