# 抖音视频下载器（Windows，不依赖微信）

粘贴抖音分享链接 → 选择保存位置 → 下载**无水印**原视频。

## 两种用法

### A. 发给朋友（免安装，推荐分发）

`dist\抖音视频下载器.zip`（约 220 MB）——对方解压到任意文件夹，双击
`DouyinDownloader.exe` 即用。**不需要装 Python 或任何东西**；有 Chrome/Edge
用系统的，没有则自动用包内自带的浏览器内核。详见包内《使用说明.txt》。

### B. 本机开发版

双击桌面「抖音视频下载器」快捷方式（指向 `douyin_gui.pyw`）。

1. 抖音 App 里复制视频链接 → 点「粘贴并识别」（整段带中文的文案也可以）
2. 选保存位置（**自动记住上次目录**，配置在 `%APPDATA%\DouyinDownloader\config.json`）
3. 开始下载。断网/中途关窗会留 `.part`，下次下同一视频**自动断点续传**。

命令行版：`python dy_extract.py "<分享链接>" [-o 输出.mp4] [--no-download]`

## 实测记录（2026-11-02，全部真实跑过）

| 测试 | 结果 |
|---|---|
| 链接1（九九·卡点舞）开发版 GUI 端到端（模拟真实鼠标键盘） | ✔ 两轮全过 |
| 链接2（歌白说·熬夜 305MB）开发版 | ✔ |
| **打包 exe**：解压到全新目录 + 干净环境变量 + 小文件下载 | ✔ |
| **打包 exe**：大文件 305.6 MB（隐藏包内内核→强制走系统 Chrome/Edge） | ✔ |
| **打包 exe**：105MB 半截 .part → 断点续传补全为完整文件 | ✔ |
| **打包 exe**：目录记忆写→重启读回 | ✔ |
| 无水印验证（抽帧检查） | ✔ |

## 原理与浏览器策略

抖音接口有两层防护：JSVMP 反爬虚拟机 + Argus 签名。本工具不实现签名算法：
Playwright 打开页面后在页面上下文 `fetch()` 抖音自己的接口，由抖音前端 JS
自动补签名；拿到无水印直链后用 curl_cffi 纯 HTTP 下载（CDN 只校验 URL 时效）。

浏览器选择顺序：系统 Chrome → 系统 Edge（App Paths 注册表探测）→ 随包内核。
打包版对通道只做一次探测、不逐个试错（Playwright channel 启动失败会残留挂死
的 node 驱动，曾因此导致任务卡死——已修复并复测）。

## 已知限制

1. 每次提取要起一次浏览器（约 8~15 秒），个人自用足够。
2. 热榜/话题链接里的数字不是视频 ID，会明确报错提示。
3. 抖音改版可能失效；仅限个人学习研究，勿二次分发。

## 本机环境注意（重要）

本机有提权运行的系统代理（`nano.exe`，127.0.0.1:65532），会让 Chromium 连接被
RESET。工具现在**自动探测**：注册表 ProxyEnable=1 且 example.com 探测失败时才加
`--proxy-server=direct://`；普通电脑不受影响、正常跟随系统网络。

## 文件

| 文件 | 说明 |
|---|---|
| `douyin_gui.pyw` | 主程序（GUI），含测试通道 `--dump-state` / `--autodl` / `--remember` |
| `dy_extract.py` | 核心库 + CLI：提取、流式下载、断点续传、代理探测 |
| `dist\抖音视频下载器.zip` | **分发件**（PyInstaller onedir + 随包 headless shell） |
| `probe\gui_e2e.py` | 开发版 GUI 端到端回归（SendInput 真实操作） |
| `probe\pkg_final.py` | 打包版终检：小文件/大文件/续传/记忆目录 |

重新打包：`python -m PyInstaller --noconfirm --clean --windowed --name DouyinDownloader --paths . --collect-all playwright --collect-all curl_cffi douyin_gui.pyw`，然后把 `%LOCALAPPDATA%\ms-playwright\{chromium_headless_shell-1248,ffmpeg-1013,winldd-1007}` 复制进 `dist\DouyinDownloader\ms-playwright\`，再压缩。
