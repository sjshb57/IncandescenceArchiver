# Twitter/X Wayback 存档工具 · archive.py

将公开 Twitter/X 账号的历史推文通过 Wayback Machine 完整存档至本地，支持 HTML、图片、视频、头像的批量抓取与增量更新，并生成可供 [IncandescenceReader](https://github.com/sjshb57/IncandescenceReader) 直接读取的索引文件。

> 此项目为 [白炽阅读器 · IncandescenceReader](https://github.com/sjshb57/IncandescenceReader) 的配套存档脚本。  
> 每个人的文字都值得被留存。

---

### 附注

> Twitter 存档组织：[TwitterArchiver](https://github.com/TwitterArchiver) 已成立
>
> 如果您想在此留档，请在[此处提交申请](https://twitterarchiver.github.io/home/guestbook.html)

---

## 项目生态

整个体系由「抓取 → 存放 → 阅读」三层构成，本仓库是最上游的抓取层。

```
                 Wayback Machine
                       │
                       ▼
           IncandescenceArchiver          ← 本仓库
           抓取快照 · 下载媒体 · 清洗 · 建索引
                       │
                       ▼
           TwitterArchiver 组织
           140+ 账号仓库 · GitHub Pages 托管
                       │
       ┌───────────────┼───────────────┐
       ▼               ▼               ▼
   网页阅读器      桌面阅读器        Android
twitterarchiver  Incandescence   TwitterArchiverApp
  .github.io        Reader
```

### 工具

| 项目 | 说明 | 技术 / 协议 |
| --- | --- | --- |
| [**IncandescenceArchiver**](https://github.com/sjshb57/IncandescenceArchiver) | 本仓库。存档工具 `archive.py`，从 Wayback CDX 抓快照、下载媒体、清洗 HTML、生成索引 | Python · AGPL-3.0 |
| [**IncandescenceReader**](https://github.com/sjshb57/IncandescenceReader) | 桌面离线阅读器。Electron 打包成免安装的便携应用，多账号切换，更新存档无需重新打包 | Electron · AGPL-3.0 |
| [**TwitterArchiverApp**](https://github.com/sjshb57/TwitterArchiverApp) | Android 客户端。原生推文流、全站视图、日期树、本地书签，另有一套存档管理工具 | Kotlin · AGPL-3.0 |

### 存档

| 仓库 | 说明 |
| --- | --- |
| [**TwitterArchiver**](https://github.com/TwitterArchiver) | 存档组织，每个账号一个独立仓库，各自托管 GitHub Pages |
| [**TwitterArchiver/home**](https://github.com/TwitterArchiver/home) | 门户与聚合数据：账号清单、全站搜索索引、跨账号回复索引 |
| [**TwitterArchiver/search**](https://twitterarchiver.github.io/home/search.html) | 网页版入口，可浏览全部账号与全站搜索 |
| [**TwitterArchiver/guestbook**](https://twitterarchiver.github.io/home/guestbook.html) | 提交想要留档的账号 |

### 相关项目

| 项目 | 说明 |
| --- | --- |
| [**webarchived_tweet_downloader**](https://github.com/gfhdhytghd/webarchived_tweet_downloader) | 另一套抓取工具，除 Wayback 外还支持通过**官方 X API** 拉取时间线、补全回复链与被回复原文，能拿到 Wayback 没存到的内容。其产出可用 `convert` 直接导入本项目格式 |

---

## 数据来源

本工具支持三条数据来源，最终都汇入同一套 `wayback_snapshots/` 结构：

| 来源 | 适用场景 | 入口命令 |
| --- | --- | --- |
| **Wayback Machine** | 公开账号，直接从互联网档案馆抓 | `fetch-cdx` → `fetch-html` → `fetch-media` |
| **webarchived_tweet_downloader 产出** | 需要 X API 补全的内容（回复链、被回复原文、Wayback 未存到的推文） | `convert <dump_dir>` |
| **X 官方数据导出包** | 自己的账号 / 锁推账号，从 X 申请的 GDPR 导出 ZIP | `import-export <export_dir>` |

---

## 目录结构

```
accounts/
  {USERNAME}/
    cdx_data.json             ← fetch-cdx 生成，Wayback CDX 快照清单
    archive.py                ← 本脚本（或从上级目录引用）
    wayback_snapshots/
      html/                   ← 下载的 Wayback HTML 快照
      json/                   ← 从 HTML 提取的推文原始 JSON
      image/                  ← 下载的图片
      video/                  ← 下载的视频
      avatar/                 ← 下载的头像
      index.json              ← build-index 生成，供前端读取
      profile.json            ← 账号信息（需手动维护）
      _log/                   ← 状态系统目录（自动生成）
        archive_index.json    ← 主账本，记录每个 URL 的下载状态
        html_done.txt
        html_failed.txt
        html_failed_all.txt
        image_done.txt
        image_failed.txt
        image_failed_all.txt
        video_*.txt
        avatar_*.txt
        media_*.txt
```

---

## 安装

需要 Python 3.10+。

```bash
pip install requests beautifulsoup4 lxml
```

---

## 使用方法

所有命令都在账号目录（`accounts/{USERNAME}/`）下执行。

### A. Wayback Machine 流程（公开账号）

```bash
# 第一步：抓取 Wayback CDX 快照清单
python ../../archive.py fetch-cdx {USERNAME}

# 第二步：下载 Wayback HTML 快照（同时提取推文 JSON）
python ../../archive.py fetch-html

# 第三步：下载图片、视频、头像
python ../../archive.py fetch-media

# 第四步：清洗 HTML 路径
python ../../archive.py clean-html

# 第五步：生成前端索引
python ../../archive.py build-index
```

后四步可以用 `all` 一把梭（不含 `fetch-cdx`）：

```bash
python ../../archive.py fetch-cdx {USERNAME}
python ../../archive.py all
```

### B. 从 webarchived_tweet_downloader 导入

[webarchived_tweet_downloader](https://github.com/gfhdhytghd/webarchived_tweet_downloader) 除 Wayback 外还能走**官方 X API**，可以补到 Wayback 没存下来的推文、完整回复链和被回复的原文。先用它抓，再导进来：

```bash
# 1. 用上游工具抓取（在它自己的目录下跑）
python download_archive.py {USERNAME}
#    产出 {USERNAME}_archive/{USERNAME}_archive_assets/{snapshots.json, json/, media/}

# 2. 回到本项目的账号目录，转换格式
python ../../archive.py convert /path/to/{USERNAME}_archive/

# 3. 从 JSON 渲染出伪 wayback HTML
python ../../archive.py render-html

# 4. 补下缺失的媒体，生成索引
python ../../archive.py fetch-media
python ../../archive.py build-index
```

`convert` 会自动在 dump 目录里定位 `snapshots.json`，兼容 `{USER}_archive/{USER}_archive_assets/` 的标准结构和手工整理过的扁平结构。

| 参数 | 说明 |
| --- | --- |
| `--include-unreferenced` | 同时复制未被任何 JSON 引用的孤立媒体（默认跳过，更贴合成品） |

### C. 从 X 官方数据导出包导入（锁推 / 私密账号）

适用于自己账号从 X 申请的完整导出包（含 `data/tweets.js`）：

```bash
python ../../archive.py import-export /path/to/twitter-{USERNAME}-export/
python ../../archive.py render-html
python ../../archive.py fetch-media
python ../../archive.py build-index
```

| 参数 | 说明 |
| --- | --- |
| `--include-deleted` | 同时收录 `deleted-tweets.js`（你主动删过的推文，默认不收） |
| `--no-fill-replied` | 不去 Wayback 补被回复 / 被引用推文的原文（默认会补） |

> `convert` 和 `import-export` 认的是不同格式：前者找 `snapshots.json`，后者找 `data/tweets.js`。传错了脚本会提示你换命令。

---

### 增量更新

直接再次执行相同命令即可，脚本会自动跳过已成功下载的内容，只处理新增和失败的部分：

```bash
python ../../archive.py fetch-cdx {USERNAME}
python ../../archive.py all
```

---

## 子命令详解

| 子命令 | 作用 |
| --- | --- |
| `fetch-cdx` | 调用 Wayback CDX API 抓取快照清单 → `cdx_data.json` |
| `fetch-html` | 下载 Wayback HTML + 抽取 JSON |
| `fetch-media` | 下载所有 JSON 里引用的媒体 |
| `fetch-image` | 只下载图片 |
| `fetch-video` | 只下载视频 |
| `fetch-avatars` | 单独重下头像（修复用） |
| `clean-html` | 清洗 HTML，重写媒体路径为本地引用 |
| `build-index` | 生成 `index.json` |
| `all` | 一把梭：`fetch-html` → `fetch-media` → `clean-html` → `build-index` |
| `convert` | 把 `download_archive.py` 的 dump 转换为本项目格式 |
| `import-export` | 把 X 官方数据导出包转换为本项目格式 |
| `render-html` | 从 JSON 渲染伪 wayback HTML（导入流程用） |
| `retry` | 重试失败项 |
| `dedup` | 媒体去重 + 孤儿清理 |
| `gen-lists` | 生成 `_url_list.txt` / `_list_media.txt` |
| `rebuild-index` | 重建 `_log/archive_index.json`（迁移 / 损坏修复） |

每个 fetch 子命令都支持 `--retry [--file PATH]`，读取失败清单只重跑失败项。

---

### `fetch-cdx`

```bash
python archive.py fetch-cdx {USERNAME} [--from YYYYMMDD] [--to YYYYMMDD] [--no-collapse-digest]
```

查询 Wayback Machine CDX API，获取账号所有推文快照的清单，写入 `cdx_data.json`。

| 参数 | 说明 |
| --- | --- |
| `--from YYYYMMDD` | 只抓指定日期之后的快照 |
| `--to YYYYMMDD` | 只抓指定日期之前的快照 |
| `--no-collapse-digest` | 不对相同内容的快照去重（默认去重） |

---

### `fetch-html`

```bash
python archive.py fetch-html [--workers N] [--delay S] [--force]
```

读取 `cdx_data.json`，逐条下载 Wayback HTML 快照，同时从 HTML 中提取推文原始 JSON 存入 `json/`。

| 参数 | 说明 |
| --- | --- |
| `--workers N` | 并发线程数（默认 7） |
| `--delay S` | 每次请求间隔秒数（默认 0.8） |
| `--force` | 强制重新下载已完成的项 |

---

### `fetch-media` / `fetch-image` / `fetch-video`

```bash
python archive.py fetch-media [--workers N] [--delay S] [--force]
```

扫描 `json/` 下所有推文 JSON，下载引用的图片和视频到 `image/` 和 `video/`，同时下载头像到 `avatar/`。

`fetch-image` 与 `fetch-video` 是只跑其中一类的窄版本，适合分阶段跑或在 CI 里拆 job。

---

### `fetch-avatars`

```bash
python archive.py fetch-avatars [--workers N] [--force] [--retry]
```

单独补全头像下载。扫描所有推文 JSON，对本地缺失的头像重新下载。

---

### `clean-html`

```bash
python archive.py clean-html
python archive.py clean-html --force
```

清洗 HTML，将媒体路径替换为本地路径：

- **头像**：直接由 pid 构造 `../avatar/avatar_{pid}.{ext}`，唯一确定
- **图片**：本地有则用真实文件名，没有则构造预期文件名（以后 retry 下到自动显示）
- **视频**：本地有则替换，没有则删除 video 标签
- **删除** Wayback Machine 工具栏和 JSON 展示元素

已清洗的 HTML 自动跳过，增量跑只处理新增的。

| 参数 | 说明 |
| --- | --- |
| `--force` | 强制重新清洗所有文件 |

---

### `build-index`

```bash
python archive.py build-index
```

扫描 `html/` 和 `json/` 目录，结合本地媒体文件，生成 `index.json` 供前端读取。

每条记录包含：

```json
{
  "file": "20240705_twitter_com_username_status_xxx.html",
  "timestamp": "2024-07-05T14:51:29.000Z",
  "date": "2024-07-05",
  "text": "推文正文预览...",
  "tweet_id": "...",
  "author_name": "...",
  "author_username": "@...",
  "author_avatar": "../avatar/avatar_xxx.jpg",
  "wanted_images": ["../image/xxx.jpg"],
  "wanted_videos": ["../video/xxx.mp4"],
  "wanted_avatars": ["../avatar/avatar_xxx.jpg"],
  "embedded_images": [],
  "embedded_videos": [],
  "is_reply": false,
  "is_virtual": false,
  "is_pinned": false
}
```

同时读取 `profile.json` 的 `pinned` 字段，把对应推文标记为 `is_pinned`。

> `profile.json` 由用户手动维护。**改完置顶后必须重跑 `build-index` 才会生效**——`is_pinned` 是建索引那一刻算出来的快照，不会自动跟着 `profile.json` 变。

---

### `render-html`

```bash
python archive.py render-html [--force]
```

从 `json/` 渲染出结构与 Wayback 快照一致的伪 HTML，供 `clean-html` 与前端使用。`convert` / `import-export` 导入后需要跑这一步，因为那两条路径只有 JSON 没有 HTML。

---

### `retry`（重试失败项）

```bash
python archive.py retry --image-failed
python archive.py retry --image-failed-all
python archive.py retry --video-failed
python archive.py retry --video-failed-all
python archive.py retry --avatar-failed
python archive.py retry --avatar-failed-all
python archive.py retry --media-failed
python archive.py retry --media-failed-all
python archive.py retry --html-failed
python archive.py retry --html-failed-all
```

从对应的 `_log/*.txt` 文件读取失败列表并重新尝试。各选项互斥，一次只能选一个。

| 后缀 | 说明 |
| --- | --- |
| `*-failed` | 可救失败（网络超时、限速等），每次增量自动重试 |
| `*-failed-all` | 永久失败（SSL 错误 = Wayback 未存档、HTTP 4xx），需手动触发 |

---

### `dedup`

```bash
python archive.py dedup [--execute] [--backup] [--delete-orphans]
```

检测重复媒体文件（相同内容不同文件名）和孤儿文件（有文件但无 JSON 引用）。默认 dry-run，加 `--execute` 实际删除。

---

### `gen-lists`

```bash
python archive.py gen-lists
```

生成 `_url_list.txt` / `_list_media.txt`，便于用外部下载器（aria2、IDM 等）批量处理。

---

### `rebuild-index`

```bash
python archive.py rebuild-index
```

从 `_log/*.txt` 和本地媒体文件重建 `archive_index.json` 状态账本。在状态文件损坏或迁移旧版本后使用。

> 注意与 `build-index` 区分：`build-index` 生成前端读取的 `wayback_snapshots/index.json`，`rebuild-index` 修复内部状态账本 `_log/archive_index.json`。

---

## 状态系统

脚本使用 `_log/` 目录追踪每个 URL 的下载状态，支持断点续传和精确重试。

| 状态 | 说明 | 对应文件 |
| --- | --- | --- |
| `done` | 下载成功 | `*_done.txt` |
| `failed` | 可救失败（超时/限速） | `*_failed.txt` |
| `failed_all` | 永久失败（未存档/4xx） | `*_failed_all.txt` |

- **SSL 错误**：Wayback Machine 对未存档资源返回 SSL 握手失败，归为永久失败，不再重试
- **ConnectTimeout**：TCP 连接超时，通常是 Wayback 限速，归为可救失败，下次增量自动重试
- **HTTP 4xx**（除 408/429）：归为永久失败

---

## GitHub Actions 自动化

提供三个 workflow 文件，fork 对应账号的仓库后开箱即用。

### `setup.yml` — 首次建档

手动触发（`workflow_dispatch`），分两阶段：

1. **第一阶段**：`fetch-html` → `build-index` → 推送（纯文字版，快速上线 Pages）
2. **第二阶段**：`fetch-media` → retry × 3 → `clean-html` → `build-index` → 推送

### `update.yml` — 每周增量

每周日 UTC 02:00（北京时间周日 10:00）自动触发，也可手动触发：

```
fetch-cdx → fetch-html → fetch-media → retry(failed) → clean-html → build-index → push
```

支持两个输入参数：

| 参数 | 说明 |
| --- | --- |
| `from_setup` | 由首次建档链式触发，跑完后自动接力全量重试 |
| `only_index` | 仅重建索引：跳过所有抓取，只跑 `build-index`。改完置顶或资料后用它快速生效，几十秒完成 |

### `retry_all.yml` — 全量重试（手动触发）

专门重试 `failed_all`（永久失败）的媒体文件，适用于 Wayback Machine 补存了之前未存档的内容：

```
retry --image-failed-all / --video-failed-all / --avatar-failed-all
→ clean-html → build-index → push
```

**仓库名即用户名**：workflow 自动从仓库名读取 Twitter/X 用户名，无需任何配置。

---

## 参数调优

修改脚本顶部的常量来调整行为：

```python
# 本地使用（默认）
REQUEST_ATTEMPTS   = 5      # 重试次数
MAX_BACKOFF        = 60.0   # 最大退避等待秒数
MEDIA_TIMEOUT_IMAGE  = (5, 40)   # (connect超时, read超时)
WAYBACK_HTML_TIMEOUT = (5, 60)
DEFAULT_WORKERS_HTML   = 7
DEFAULT_WORKERS_MEDIA  = 8
DEFAULT_WORKERS_AVATAR = 4
DEFAULT_DELAY_HTML     = 0.8
DEFAULT_DELAY_MEDIA    = 0.3

# GitHub Actions 推荐（数据中心网络，IP 每次不同）
REQUEST_ATTEMPTS   = 1
MAX_BACKOFF        = 20.0
MEDIA_TIMEOUT_IMAGE  = (4, 8)
DEFAULT_WORKERS_HTML   = 30
DEFAULT_WORKERS_MEDIA  = 40
DEFAULT_WORKERS_AVATAR = 30
DEFAULT_DELAY_HTML     = 0.15
DEFAULT_DELAY_MEDIA    = 0.08
```

---

## License

Copyright © 2026 sjshb57

本项目基于 [GNU Affero General Public License v3.0](https://www.gnu.org/licenses/agpl-3.0.html) 开源。你可以自由使用、修改和分发本项目，但衍生作品必须同样以 AGPL-3.0 协议开源。

存档内容本身的版权归原作者所有。本工具仅做数字保存，不主张任何内容权利。

---

## 赞助

![赞助图片](https://free.picui.cn/free/2026/06/24/6a3b25866f0fd.jpg)
