# OneAPI Gateway (Go 版本)

> **[English](./README.md)** | **中文**

高性能聚合 LLM API 网关，从原 Python 版本用 Go 完全重写。将多个上游供应商（OpenAI、Anthropic、Gemini、DeepSeek、Azure 等）聚合为一个统一的标准 OpenAI 兼容接口，提供系统托盘 + AiTun + APIShare 隧道集成。

**版本**：2.0.0
**语言**：Go 1.23+
**许可证**：MIT

---

## ✨ 为什么用 Go？

原 Python 版本（[备份在 ctz168/oneapi_py](https://github.com/ctz168/oneapi_py)）运行良好，但有一个痛点：**PyInstaller 不能跨平台编译**。要生成 Windows `.exe`、macOS binary、Linux binary，要么在每台机器上分别构建，要么维护复杂的 GitHub Actions 矩阵构建。

Go 优雅地解决了这个问题——单条 `go build` 命令带 `GOOS=windows GOARCH=amd64` 就能在 Linux 上生成 Windows binary，零 CGo 依赖。三个平台在任何机器上都能几秒内构建出来。

## ✨ 核心特性

- **单文件静态 binary** — 无需安装 Python，无共享库依赖，每平台约 17 MB
- **真正的跨平台编译** — `GOOS=windows GOARCH=amd64 go build` 从 Linux 也能产出 Windows binary（反之亦然）
- **系统托盘** — Windows 用 `getlantern/systray`（真托盘图标）。Linux/macOS 跑 headless（仅 HTTP 服务）。
- **万能模型别名 (`mymodel`)** — 在所有活跃渠道间自动负载均衡路由
- **多供应商聚合** — OpenAI、Anthropic、Gemini、DeepSeek、Azure 一个接口搞定
- **每 Key 限速** — RPM + 总 token 配额
- **每日自动重置** — 额度耗尽自动熔断 + 每日 00:00 自动恢复
- **路径兼容** — `/v1/chat/completions` 与 `/chat/completions` 均支持
- **AiTun 集成** — 粘贴 `ait_xxx` secret key，自动启动隧道 + 发现公网域名
- **APIShare 出售 Token** — 一键将 `mymodel` 发布到 apishare.cc 市场
- **数据库兼容** — 直接读 Python 创建的 `oneapi.db`（SQLite schema 完全一致）

## 🚀 快速开始

### 方式 1：下载发布包

前往 [最新发布页](https://github.com/samaidev/oneapi_r/releases/latest) 下载对应平台的 binary：
- `oneapi-linux-amd64` — Linux x86_64（约 17 MB）
- `oneapi-windows-amd64.exe` — Windows x86_64（约 18 MB）
- `oneapi-darwin-arm64` — macOS Apple Silicon（约 16 MB）

运行：
```bash
chmod +x oneapi-linux-amd64
./oneapi-linux-amd64    # 启动在 http://localhost:3000
```

### 方式 2：从源码构建

```bash
git clone https://github.com/samaidev/oneapi.git
cd oneapi
go build -o oneapi-gateway ./cmd/oneapi-gateway
./oneapi-gateway
```

### 方式 3：交叉编译全平台

```bash
# 在任何平台上都能产出 3 个 binary
GOOS=linux GOARCH=amd64 go build -o dist/oneapi-linux-amd64 ./cmd/oneapi-gateway
GOOS=windows GOARCH=amd64 go build -o dist/oneapi-windows-amd64.exe ./cmd/oneapi-gateway
GOOS=darwin GOARCH=arm64 CGO_ENABLED=0 go build -o dist/oneapi-darwin-arm64 ./cmd/oneapi-gateway
```

就这样。无需 Docker、无需 PyInstaller spec、无需 GitHub Actions 矩阵、无需工具链折腾。

## 🛠️ 配置

编辑 `.env`：

```env
PORT=3000
DATABASE_URL=sqlite+aiosqlite:///./oneapi.db
AITUN_SECRET_KEY=    # 可选: ait_xxx 来自 https://aitun.cc/ 用于获取公网域名
```

或用浏览器访问 `http://localhost:3000/config`（Web 配置面板）。

### AiTun 公网域名

在 [aitun.cc](https://aitun.cc/) 登录、注册子域名、复制 `ait_xxx` secret key，粘贴到 `.env` 或 Config Panel。OneAPI 自动启动 `aitun -k <key> -p <port>` 并发现公网域名（约 5 秒）。

### 渠道（上游供应商）

1. 打开 `http://localhost:3000`（默认管理员密码 `admin123`）
2. 添加渠道（DeepSeek、Claude、OpenAI 等）
3. 配置模型映射 — `model: "mymodel"` 自动路由到任意活跃渠道

## 💰 出售 Token 到 APIShare

1. 在 [apishare.cc/dashboard](https://apishare.cc/dashboard) 获取 APIShare agent secret key（`apsagt-...`）
2. 浏览器打开 `http://localhost:3000/sell-token`
3. 粘贴 key + 点击 "OK & Start Selling"
4. OneAPI 自动创建本地 token + 注册隧道 provider + 启动 `apishare-tunnel` + 激活

停止：在同一页面点击 "Stop selling"。

## 📖 API 用法

OneAPI 是 OpenAI 的直接替换：

```bash
curl http://localhost:3000/v1/chat/completions \
  -H "Authorization: Bearer sk-your-oneapi-token" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "mymodel",
    "messages": [{"role": "user", "content": "你好！"}]
  }'
```

## 🧱 架构

```
你的应用 / OpenAI SDK
    │  Bearer sk-oneapi-...
    ▼
┌───────────────────────────────────────┐
│  OneAPI Gateway (Go binary)          │
│  - chi router + CORS                 │
│  - balancer (mymodel 路由)            │
│  - proxy (上游转发 + SSE 流式)        │
│  - rate_limiter + adaptive_limiter    │
│  - reasoning_normalizer              │
│  - i18n (embed JSON, en + zh)         │
│  - tray (Windows 系统托盘)            │
│  - aitun (隧道自动启动)               │
│  - apishare (sell token 流程)         │
└───────────────────────────────────────┘
    │  model=mymodel (负载均衡)
    ▼
  OpenAI / Claude / Gemini / DeepSeek / Azure

可选公网访问:
  ┌───────────────────────────────────────┐
  │  aitun.cc 隧道 (自动启动)              │
  │  → myapp.t.aitun.cc                   │
  └───────────────────────────────────────┘
```

## 📁 项目结构

```
oneapi/
├── cmd/oneapi-gateway/main.go    # 入口
├── internal/
│   ├── config/                   # .env + 环境变量加载
│   ├── db/                       # sqlx + SQLite (modernc.org/sqlite, 无 CGo)
│   ├── models/                   # Channel, ApiToken, RequestLog 等
│   ├── balancer/                 # mymodel 路由 + 午夜重置
│   ├── limiter/                  # RateLimiter + AdaptiveLimiter
│   ├── proxy/                    # 上游转发 + reasoning 标准化 + utils
│   ├── handlers/                 # OpenAI + admin API (25+ 端点)
│   ├── tray/                     # 系统托盘 (仅 Windows)
│   ├── aitun/                    # AiTun 隧道自动启动
│   ├── apishare/                 # APIShare sell-token 流程 + Web UI
│   └── i18n/                     # embed JSON 翻译 (en + zh)
├── web/
│   ├── static/                   # 前端 HTML/JS/CSS (复用自 Python 版)
│   └── web.go                    # embed.FS 静态资源
├── .env.example
├── go.mod / go.sum
└── README.md
```

## 🔧 故障排除

- **Linux/macOS 无托盘图标** — 设计如此。Linux 需 `libayatana-appindicator3-dev`（用 `-tags=appindicator` 构建），macOS 需 Cocoa/CGo。开箱即用时这两个平台跑 headless — 直接浏览器访问 `http://localhost:3000`。
- **数据库错误 "out of memory"** — 确保 `.env` 有 `DATABASE_URL=sqlite+aiosqlite:///./oneapi.db`（或绝对路径）。
- **无法登录** — 默认管理员密码 `admin123`。首次登录后请修改。
- **AiTun 不启动** — 确保 `aitun-client-*` binary 在 PATH 或 OneAPI binary 旁。从 [aitun.cc/downloads](https://aitun.cc/downloads) 下载。

## 🔗 链接

- **源码**：[samaidev/oneapi](https://github.com/samaidev/oneapi)
- **公开发布**：[samaidev/oneapi_r](https://github.com/samaidev/oneapi_r/releases/latest)
- **Python 备份**：[ctz168/oneapi_py](https://github.com/ctz168/oneapi_py)
- **APIShare**：[apishare.cc](https://apishare.cc)
- **AiTun**：[aitun.cc](https://aitun.cc)
