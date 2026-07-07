<div align="center">

# 🐄 MooSaver

**一鍵批次備份 Moodle 課程資源的自架下載工具**

*Batch-download and organise all your Moodle course files from a single self-hosted web app.*

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.12%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Django](https://img.shields.io/badge/Django-6.0-092E20?logo=django&logoColor=white)](https://www.djangoproject.com/)
[![Vue.js](https://img.shields.io/badge/Vue.js-3-4FC08D?logo=vuedotjs&logoColor=white)](https://vuejs.org/)
[![Docker](https://img.shields.io/badge/Docker-ready-2496ED?logo=docker&logoColor=white)](https://hub.docker.com/r/0blueyan0/moosaver)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#-貢獻)

</div>

---

MooSaver 是一個基於 **Django + Vue.js** 的 Moodle 資源下載工具。輸入學號與密碼後，它會透過 Moodle 官方 Mobile App API 取得存取權杖，批次抓取你所有課程的講義、作業、公告等資源，並自動按課程分類、集中管理，讓你不必再一個一個檔案手動點下載。

> 底層下載引擎採用開源專案 [Moodle-DL](https://github.com/C0D3D3V/Moodle-DL)，並包裝成友善的多使用者 Web 介面。

## 📖 目錄

- [功能特色](#-功能特色)
- [支援平台](#-支援平台)
- [技術架構](#-技術架構)
- [快速開始](#-快速開始)
  - [使用 Docker（推薦）](#使用-docker推薦)
  - [本地開發](#本地開發)
- [設定說明](#-設定說明)
- [使用方法](#-使用方法)
- [專案結構](#-專案結構)
- [安全性說明](#-安全性說明)
- [Roadmap](#-roadmap)
- [貢獻](#-貢獻)
- [開源致謝](#-開源致謝)
- [授權條款](#-授權條款)

## ✨ 功能特色

- 🚀 **一鍵批次下載** — 自動抓取所有課程的資源，不需逐一手動點擊
- 📁 **智能整理** — 依課程自動分類建立資料夾，檔案結構清晰
- 🗂️ **線上檔案管理** — 內建檔案樹瀏覽器，可預覽、單檔下載或整包打包成 ZIP
- 🔍 **檔案搜尋** — 快速搜尋與定位下載後的檔案
- 👥 **多使用者支援** — 每位使用者的資源獨立隔離、獨立管理
- 📊 **下載統計** — 記錄下載次數與總容量，管理員可檢視全站統計
- 🎨 **響應式介面** — 桌面與行動裝置皆適用

## 🌐 支援平台

| 平台 | 網址 | 狀態 |
|------|------|------|
| 逢甲大學 iLearn | `https://ilearn.fcu.edu.tw/` | ✅ 內建 |
| 其他 Moodle 站台 | 自訂網址 | ✅ 支援（於下載頁選擇「自訂網址」） |

> 理論上支援任何開啟了 `moodle_mobile_app` Web Service 的 Moodle 站台。

## 🛠 技術架構

| 層級 | 技術 |
|------|------|
| 後端 | Django 6.0 · Python 3.12+ |
| 前端 | Vue.js 3（CDN 引入）· Font Awesome · Noto Sans TC |
| 資料庫 | SQLite |
| 下載核心 | [Moodle-DL](https://github.com/C0D3D3V/Moodle-DL) 2.3.13 |
| 部署 | Docker / Docker Compose |

## 🚀 快速開始

### 使用 Docker（推薦）

> ⚠️ 官方 Image `0blueyan0/moosaver:latest` **僅提供 x86（amd64）平台**。若使用 Apple Silicon 或 ARM 主機，請改用[本地開發](#本地開發)方式，或自行 `docker build`。

**快速啟動：**

```bash
docker run -d \
  --name moosaver \
  -p 8000:8000 \
  -v /path/to/your/downloads:/app/users_data \
  -e DATABASE_PATH=/app/users_data/db.sqlite3 \
  0blueyan0/moosaver:latest
```

啟動後開啟瀏覽器造訪 <http://localhost:8000> 即可。

**使用 Docker Compose：**

建立 `docker-compose.yml`：

```yaml
services:
  moosaver:
    image: 0blueyan0/moosaver:latest
    ports:
      - "8000:8000"
    volumes:
      - /path/to/your/downloads:/app/users_data
    environment:
      - DATABASE_PATH=/app/users_data/db.sqlite3
    restart: unless-stopped
```

執行：

```bash
docker compose up -d
```

> 容器首次啟動時會自動執行資料庫遷移，並建立預設管理員帳號（見[使用方法](#-使用方法)）。

### 本地開發

需要 **Python 3.12+**。

```bash
# 1. 取得原始碼
git clone https://github.com/Ruxiu0409/moosaver.git
cd moosaver

# 2. 建立並啟用虛擬環境（建議）
python3 -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

# 3. 安裝依賴
pip install -r requirements.txt

# 4. 執行資料庫遷移（會在 repo 根目錄建立 db.sqlite3）
python manage.py migrate

# 5. 建立管理員帳號
python manage.py createsuperuser

# 6. 啟動開發伺服器
python manage.py runserver
```

啟動後造訪 <http://127.0.0.1:8000>。

## ⚙️ 設定說明

### Docker 環境變數

| 參數 | 說明 | 範例 |
|------|------|------|
| `-p 8000:8000` | 埠號對應（主機:容器），可改為 `-p 3000:8000` | 主機以 3000 埠存取 |
| `-v .../downloads:/app/users_data` | 資料卷掛載，下載檔案會存到主機此路徑 | 保留下載成果 |
| `-e DATABASE_PATH=/app/users_data/db.sqlite3` | 資料庫檔案路徑 | 不清楚用途請保持原樣 |

### 下載內容選項

每次下載會依課程建立設定，預設抓取以下項目（可於 `download/views.py` 調整）：

| 選項 | 預設 | 選項 | 預設 |
|------|:---:|------|:---:|
| 作業繳交 (submissions) | ✅ | 測驗 (quizzes) | ✅ |
| 說明文字 (descriptions) | ✅ | 工作坊 (workshops) | ✅ |
| 討論區 (forums) | ✅ | 資料庫 (databases) | ❌ |
| 課程 (lessons) | ❌ | 書本 (books) | ❌ |
| 行事曆 (calendars) | ❌ | 說明中的連結檔案 | ❌ |

## 📚 使用方法

1. **登入** — 註冊新帳號，或使用預設管理員帳號登入：

   | 帳號 | 密碼 |
   |------|------|
   | `admin` | `admin123` |

   > 🔐 **正式使用前請務必修改預設管理員密碼。**

2. **輸入 Moodle 憑證** — 在下載頁面填入你的 Moodle 學號與密碼。
3. **選擇平台** — 從清單選擇 Moodle 站台，或輸入自訂網址。
4. **開始下載** — 點擊「開始下載」，進度會即時顯示。
5. **管理檔案** — 於檔案管理頁面瀏覽、下載單檔，或將整個課程打包成 ZIP。

## 🗂 專案結構

```
moosaver/
├── download/            # 下載功能應用（下載、進度回報、檔案管理）
│   ├── moodle_dl_utils.py   # Moodle-DL 封裝與下載流程
│   ├── views.py             # 下載 / 檔案管理 API 與頁面
│   └── templates/           # 下載、檔案管理等前端模板
├── users/               # 使用者管理應用（註冊、登入、管理後台）
├── moosaver/            # Django 專案設定（settings、urls、wsgi/asgi）
├── requirements.txt     # Python 依賴（已鎖定版本）
├── Dockerfile           # Docker 映像檔建置設定
├── entrypoint.sh        # 容器啟動腳本（遷移 + 建立 admin + 啟動）
├── manage.py            # Django 管理指令
└── db.sqlite3           # SQLite 資料庫（執行後自動產生）
```

## 🔒 安全性說明

- 你輸入的 **Moodle 密碼不會被儲存**；系統僅用它向 Moodle 官方 API 換取存取權杖（token），權杖存放於你自己的資料卷中。
- 本專案預設以 Django 開發模式（`DEBUG=True`）執行，**不適合直接對外公開部署**。若要架設於公網，請自行關閉 DEBUG、更換 `SECRET_KEY`、設定 `ALLOWED_HOSTS`，並置於反向代理之後。
- 建議在信任的本機或內網環境使用。

## 🗺 Roadmap

- [ ] 提供多架構（arm64）Docker 映像檔
- [ ] 排程 / 自動定期同步
- [ ] 下載完成通知（Email / Telegram / Discord）
- [ ] 更完善的下載選項 UI（免改程式碼即可調整）

歡迎在 [Issues](https://github.com/Ruxiu0409/moosaver/issues) 提出想法。

## 🤝 貢獻

歡迎任何形式的貢獻！

1. Fork 本專案
2. 建立特性分支：`git checkout -b feature/my-feature`
3. 提交變更：`git commit -m "Add my feature"`
4. 推送分支：`git push origin feature/my-feature`
5. 開啟 Pull Request

有問題或建議也歡迎直接建立 [Issue](https://github.com/Ruxiu0409/moosaver/issues)。

## 🙏 開源致謝

本專案站在以下優秀開源專案的肩膀上：

| 專案 | 用途 | 授權 |
|------|------|------|
| [Moodle-DL](https://github.com/C0D3D3V/Moodle-DL) | Moodle 下載核心 | GPL-3.0 |
| [Django](https://github.com/django/django) | Python Web 框架 | BSD-3-Clause |
| [Vue.js 3](https://github.com/vuejs/core) | 前端框架 | MIT |
| [Font Awesome](https://github.com/FortAwesome/Font-Awesome) | 圖示字體庫 | Font Awesome Free |
| [Noto Sans TC](https://fonts.google.com/noto/specimen/Noto+Sans+TC) | 繁體中文字體 | SIL Open Font License |

## 📄 授權條款

本專案採用 [GNU General Public License v3.0](LICENSE) 授權。

由於使用了以 GPL-3.0 授權的 Moodle-DL，本專案亦遵循 GPL-3.0。

---

<div align="center">
若這個專案對你有幫助，歡迎點一顆 ⭐ Star 支持！
</div>
