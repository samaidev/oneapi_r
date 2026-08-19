# OneAPI Gateway

> **[English](./README.md)** · **[中文](./README.zh-CN.md)**

OneAPI Gateway 是基于 Python 的高性能 LLM API 聚合转发服务。它将多个上游供应商（OpenAI、Anthropic、Gemini、DeepSeek、Azure 等）聚合在一个统一的标准 OpenAI 兼容接口下，提供 Windows 系统托盘管理工具与 [APIShare](https://apishare.cc) 一键集成。

**版本**：1.0.0  
**许可证**：MIT  
**状态**：生产就绪

---

## ✨ 核心特性

- **系统托盘管理** — 启动、停止、重启、配置，以及一键"Sell token..."将闲置算力出售给 APIShare —— 全部在 Windows 系统托盘完成。
- **万能模型别名 (`mymodel`)** — 调用任意模型时使用 `mymodel`，OneAPI 自动路由到最便宜/最健康的活跃渠道 —— 真正的负载均衡。
- **多供应商聚合** — OpenAI、Anthropic、Gemini、DeepSeek、Azure 等多家供应商，统一在 OpenAI 兼容接口下。
- **每日自动重置与测试** — 智能额度耗尽检测 + 自动熔断。每日 12:05 渠道健康检查。每日 00:00 自动重置配额。
- **每 Key 限速** — 生成自定义 API Key，可设置每分钟 RPM 限制与总配额。
- **路径兼容** — 同时支持 `/v1/chat/completions` 与 `/chat/completions` —— OpenAI SDK 直接替换。
- **一键接入 APIShare** — 右键托盘 → "Sell token..." → 粘贴你的 APIShare agent token → OneAPI 自动创建本地 token、将 `mymodel` 注册为隧道模式 provider、启动 APIShare 隧道客户端、并激活。每次调用获得 90% 收入。

---

## 🚀 快速开始

### 方式 1：下载发布包

1. 前往 [最新发布页](https://github.com/samaidev/oneapi_r/releases/latest)。
2. 下载匹配你平台的包：
   - `oneapi-1.0.0-windows.zip` — Windows（含 `start.bat`）
   - `oneapi-1.0.0-linux.tar.gz` — Linux/macOS（含 `start.sh`）
   - `oneapi-1.0.0-source.zip` — 跨平台源码
3. 解压并运行：
   - **Windows**：双击 `start.bat`
   - **Linux/macOS**：`./start.sh`
4. 托盘图标出现，服务在 `http://localhost:3000` 启动。

### 方式 2：从源码运行

```bash
git clone https://github.com/samaidev/oneapi.git
cd oneapi
pip install -r requirements.txt -r tray-requirements.txt
python app.py        # 无界面模式
# 或
python tray_manager.py   # 带托盘（推荐 Windows）
```

---

## 🛠️ 配置

### 服务端口

编辑 `.env`：

```env
PORT=3000
DATABASE_URL=sqlite+aiosqlite:///./oneapi.db
DUCKDNS_DOMAIN=
DUCKDNS_TOKEN=
```

应用：右键托盘 → **重启服务**。

### 渠道（上游供应商）

1. 右键托盘 → **打开管理面板**（或访问 `http://localhost:3000`）。
2. 默认管理员密码：`admin123`（首次登录后请修改）。
3. 添加渠道（DeepSeek、Claude、OpenAI、Gemini 等）。
4. 配置模型映射：
   - 格式：`别名:真实模型名`（例如 `pro-chat:gpt-4o`）
   - **万能模式**：请求 `model: "mymodel"` 会自动路由到任意活跃渠道，并使用其首个模型。APIShare 发布的就是这个。

### API Tokens

在管理面板 → **Tokens** 标签：
- 创建每个客户端的 token，设置 `max_rpm` 与 `max_tokens` 限制。
- "Sell token..." 功能会自动创建名为 `apishare-cc-public` 的 token，用于 APIShare 公网流量。

---

## 💰 出售 Token 到 APIShare（一键）

将本地 `mymodel` 变现的最快方式：

1. **获取 APIShare agent token**：登录 [apishare.cc/dashboard](https://apishare.cc/dashboard) → Agent Tokens 卡片 → 创建带 `provider:write` 权限（或 `*` 全权限）的 token。复制 `apsagt-...` 值。
2. **打开对话框**：右键 OneAPI 托盘图标 → **Sell token...**。
3. **粘贴 agent token** 到输入框。如有需要可调整本地端口（默认 3000）和模型名（默认 `mymodel`）。
4. **点击"OK & Start Selling"**。OneAPI 会：
   - 调用 `https://apishare.cc/api/user/agent-tokens` 验证 agent token。
   - 在本地创建名为 `apishare-cc-public` 的 OneAPI token（APIShare 公网调用方将使用它）。
   - 在 APIShare 上注册一个隧道模式 provider，`model_name=mymodel` 且 `api_key=<本地token>`。
   - 通过 pip 安装 `apishare-tunnel`（若未安装），然后作为后台子进程启动 `apishare -k <tunnelToken> -p 3000`。
   - 等待隧道连接，然后调用 `POST /api/provider/activate/{api_id}` 激活。
5. **完成。** `mymodel` 现在可通过 `https://apishare.cc/v1/chat/completions`（或 APIShare 市场页面）公网调用。每次调用获得 90% 收入。

> 💡 **对话框内超链接**：输入区有可点击的"打开 apishare.cc/dashboard"链接，方便你中途获取 agent token。

### 鉴权握手流程

- APIShare 消费方发送 `Authorization: Bearer <apishare_consumer_key>` 到 apishare.cc。
- APIShare 校验消费方 key、扣余额，然后通过独立隧道服务（端口 3397）转发到本地 apishare-tunnel 客户端。
- APIShare 平台注入 `Authorization: Bearer <你的本地 OneAPI token>`（注册 provider 时存储的），让本地 OneAPI 能鉴权转发请求。
- OneAPI 将请求路由到 `mymodel` —— 也就是在你所有活跃上游渠道间负载均衡。

---

## 📖 API 使用

OneAPI 运行在 `http://localhost:3000`，是 OpenAI 的直接替换：

```bash
curl http://localhost:3000/v1/chat/completions \
  -H "Authorization: Bearer sk-your-oneapi-token" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "mymodel",
    "messages": [{"role": "user", "content": "你好！"}]
  }'
```

`/v1/chat/completions` 与 `/chat/completions` 都支持。

---

## 🧱 架构

```
                          ┌──────────────────────────────┐
                          │  你的应用 / OpenAI SDK        │
                          └────────────┬─────────────────┘
                                       │ Bearer sk-oneapi-...
                                       ▼
                          ┌──────────────────────────────┐
                          │  OneAPI Gateway (Python)      │
                          │  - aiohttp + SQLAlchemy       │
                          │  - pystray (Windows 托盘)     │
                          │  - sell_token_panel.py        │
                          │    └─ 创建本地 token          │
                          │    └─ 注册到 APIShare          │
                          │    └─ 启动 apishare-tunnel    │
                          └────────────┬─────────────────┘
                                       │ model=mymodel (负载均衡)
                                       ▼
                  ┌─────────────┬──────┴──────┬─────────────┐
                  ▼             ▼             ▼             ▼
            ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
            │ OpenAI  │  │Claude   │  │ Gemini  │  │ DeepSeek│
            └─────────┘  └─────────┘  └─────────┘  └─────────┘

                          ┌──────────────────────────────┐
                          │  apishare.cc (Go 平台)        │
                          │  - 消费方请求                 │
                          │  - 通过隧道转发 (3397)         │
                          └────────────┬─────────────────┘
                                       │ WSS 隧道 + Bearer <本地token>
                                       ▼
                          ┌──────────────────────────────┐
                          │  apishare-tunnel 客户端       │
                          │  (Python, 托盘自动启动)       │
                          └────────────┬─────────────────┘
                                       │ HTTP localhost:3000
                                       ▼
                          ┌──────────────────────────────┐
                          │  OneAPI Gateway               │
                          └──────────────────────────────┘
```

---

## 📁 项目结构

```
oneapi/
├── app.py                  # 主 aiohttp 服务（端口 3000）
├── tray_manager.py         # Windows 系统托盘管理器
├── sell_token_panel.py    # APIShare 集成对话框（新）
├── config_panel.py        # .env 编辑对话框（Tkinter）
├── i18n.py                # 英文（默认）+ 中文 翻译
├── admin.py               # 管理 API（登录、渠道、token、日志）
├── handlers.py            # /v1/chat/completions, /v1/models
├── core/
│   ├── load_balancer.py   # mymodel 路由逻辑
│   ├── proxy.py           # 上游供应商代理
│   ├── database.py        # SQLAlchemy 模型
│   └── ...
├── static/                # 管理面板 UI
├── requirements.txt
├── tray-requirements.txt
├── start.bat              # Windows 启动器
├── start.sh               # Linux/macOS 启动器
└── launcher-select.bat   # 选择启动模式
```

---

## 🔧 故障排除

- **Linux 上无托盘图标**：安装 `python3-tk` + `libappindicator3-dev`，然后重装 `pip install pystray`。
- **"服务已启动但未检测到"**：查看 `service.log` 的上游错误。
- **"apishare-tunnel: command not found"**：Sell token 对话框会通过 pip 自动安装；若仍失败，手动执行 `pip install apishare-tunnel`。
- **激活时"Tunnel not connected"警告**：隧道注册可能需要超过 8 秒。等待 30 秒后再次点击 **Sell token...** —— OneAPI 会检测现有状态并重试激活。也可在 apishare.cc 管理面板手动激活。
- **本地管理员登录失败**：若你已将默认密码从 `admin123` 修改，Sell token 对话框无法自动创建本地 token。请改回默认，或在管理面板手动创建名为 `apishare-cc-public` 的 token 并使用。

---

## 📜 许可证

MIT。详见 [LICENSE](./LICENSE)。

## 🙏 致谢

- [APIShare](https://apishare.cc) — 隧道服务 + 市场
- [aiohttp](https://docs.aiohttp.org/) — 异步 HTTP 服务器
- [pystray](https://github.com/moses-palmer/pystray) — 跨平台托盘
- [Pillow](https://python-pillow.org/) — 图标渲染

## 🔗 链接

- **私有源码仓库**：[samaidev/oneapi](https://github.com/samaidev/oneapi)
- **公开发布仓库**：[samaidev/oneapi_r](https://github.com/samaidev/oneapi_r/releases/latest)
- **APIShare 网站**：[apishare.cc](https://apishare.cc) · [/oneapi](https://apishare.cc/oneapi) 介绍页
