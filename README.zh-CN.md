# OneAPI Gateway

> **[English](./README.md)** | 中文

OneAPI Gateway 是高性能聚合 LLM API 网关，将 OpenAI、Anthropic、Gemini、DeepSeek、Azure 等多个上游供应商整合到一个统一的 OpenAI 兼容端点之后——单一静态二进制，内置系统托盘管理器与管理后台。本仓库只发布可直接运行的成品包；源代码位于开发仓库 [samaidev/oneapi](https://github.com/samaidev/oneapi)。

---

## ✨ 功能特性

- **单文件静态二进制** — 无运行时、无依赖，下载即用（每平台约 17–19 MB）
- **多供应商聚合** — OpenAI、Anthropic、Gemini、DeepSeek、Azure 等统一到一个 OpenAI 兼容接口
- **万能模型别名（`mymodel`）** — 用 `mymodel` 调用任意模型，请求在所有活跃通道间自动负载均衡
- **系统托盘管理器** — 托盘图标直接启停、重启、配置、出售闲置额度（Windows 真托盘、macOS 菜单栏图标、Linux 无界面运行）
- **内置管理后台** — 通道、令牌、日志、配置一页搞定（`http://localhost:3000`）
- **每 Key 限速** — 自定义 API Key 支持 RPM 限速与总 token 配额上限
- **每日自动重置与健康检查** — 额度耗尽自动熔断，每日 00:00 自动恢复，每日定时通道测试
- **免费模型自动加载** — 内置 AI 智能体启动时自动同步 opencode.ai 最新免费模型，无需任何 API Key 开箱即用
- **路径兼容** — `/v1/chat/completions` 与 `/chat/completions` 均支持（OpenAI SDK 无缝替换）
- **AiTun 集成** — 粘贴 `ait_xxx` 密钥自动开通公网域名
- **APIShare 一键出售** — 一键把 `mymodel` 发布到 apishare.cc，闲置额度变现

## 📥 下载

前往 **[Releases 发布页](https://github.com/samaidev/oneapi_r/releases/latest)** 下载最新版本（当前：**v2.0.1**）：

| 文件 | 平台 | 大小 |
|---|---|---|
| [`oneapi-windows-amd64.exe`](https://github.com/samaidev/oneapi_r/releases/download/v2.0.1/oneapi-windows-amd64.exe) | Windows x86_64 | ~18 MB |
| [`OneAPI-v2.0.1.dmg`](https://github.com/samaidev/oneapi_r/releases/download/v2.0.1/OneAPI-v2.0.1.dmg) | macOS（OneAPI.app） | ~12 MB |
| [`OneAPI-v2.0.1-macOS.zip`](https://github.com/samaidev/oneapi_r/releases/download/v2.0.1/OneAPI-v2.0.1-macOS.zip) | macOS（压缩包） | ~11 MB |
| [`oneapi-linux-amd64`](https://github.com/samaidev/oneapi_r/releases/download/v2.0.1/oneapi-linux-amd64) | Linux x86_64 | ~18 MB |
| [`oneapi-darwin-amd64`](https://github.com/samaidev/oneapi_r/releases/download/v2.0.1/oneapi-darwin-amd64) | macOS 裸二进制 | ~19 MB |

## 🚀 安装与运行

### Windows

1. 下载 `oneapi-windows-amd64.exe`（上方按钮或 [发布页](https://github.com/samaidev/oneapi_r/releases/latest)）。
2. 双击运行，无需安装。
3. 浏览器打开 http://localhost:3000 ，默认密码 `admin123`。

### macOS & Linux（一键安装，推荐）

在终端粘贴执行：

```bash
curl -fsSL https://apishare.cc/oneapi-install.sh | bash
```

脚本自动下载对应安装包、完成安装、配置开机自启并在后台启动 OneAPI：

- **macOS**：安装到 `/Applications/OneAPI.app` + LaunchAgent（菜单栏出现托盘图标）
- **Linux**：安装到 `~/.local/share/oneapi` + systemd 用户服务

安装完成后浏览器打开 http://localhost:3000 ，默认密码 `admin123`。

### 手动安装（一键脚本不可用时）

**第 1 步 —— 安装 App / 二进制**

- **macOS**：下载 `.dmg`，挂载后把 `OneAPI.app` 拖入「应用程序」。
- **Linux**：下载 `oneapi-linux-amd64`，然后：

```bash
mkdir -p ~/oneapi && cd ~/oneapi
chmod +x oneapi-linux-amd64
./oneapi-linux-amd64
```

**第 2 步 —— 仅 macOS：解除隔离属性（只需一次）**

```bash
xattr -dr com.apple.quarantine /Applications/OneAPI.app
```

**第 3 步 —— macOS 终端启动**

```bash
mkdir -p ~/oneapi && cd ~/oneapi && /Applications/OneAPI.app/Contents/MacOS/oneapi-gateway
```

不要关闭终端窗口（关闭即停止 OneAPI）。托盘图标显示 "OneAPI - Running"。需要开机自启请优先使用上面的一键安装脚本。

## 🖥️ 首次使用

1. 浏览器打开 **http://localhost:3000** 登录（默认密码 `admin123`，登录后请修改）。
2. 在后台添加**通道**（DeepSeek、Claude、OpenAI 等）。
3. 用统一端点 + `model: "mymodel"` 调用任意模型，请求自动路由到最健康的活跃通道：

```bash
curl http://localhost:3000/v1/chat/completions \
  -H "Authorization: Bearer sk-your-oneapi-token" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "mymodel",
    "messages": [{"role": "user", "content": "你好！"}]
  }'
```

内置侧边栏 AI 智能体开箱即用——每次启动自动同步 opencode.ai 最新免费模型（无需 API Key）。

## 💰 在 APIShare 出售闲置额度

1. 在 [apishare.cc/dashboard](https://apishare.cc/dashboard) 获取代理密钥（`apsagt-...`）。
2. 右键 OneAPI 托盘图标 → **Sell token...**（或打开 `http://localhost:3000/sell-token`）。
3. 粘贴密钥并点击 **OK & Start Selling** —— OneAPI 自动启动隧道、把你的 `mymodel` 发布到 apishare.cc 并开始承接付费流量，每次调用你分成 90%。在同一页面点击 **Stop selling** 即可停止。

## ❓ 常见问题

- **无法登录** —— 默认管理员密码 `admin123`，首次登录后请修改。
- **Linux/macOS 没有托盘图标** —— Linux 默认无界面运行（用浏览器打开后台即可）；macOS 菜单栏图标需将 App 安装到「应用程序」目录。
- **macOS 提示"无法验证开发者"** —— 执行一次 `xattr -dr com.apple.quarantine /Applications/OneAPI.app`。
- **3000 端口被占用** —— 编辑二进制同目录的 `.env` 设置 `PORT=<其他端口>`（或在 `http://localhost:3000/config` 配置页修改）后重启。

## 🔗 相关链接

- **安装指南与文档**：https://apishare.cc/oneapi
- **源码（开发仓库）**：[samaidev/oneapi](https://github.com/samaidev/oneapi)
- **发布下载**：[samaidev/oneapi_r/releases/latest](https://github.com/samaidev/oneapi_r/releases/latest)
- **APIShare**：[apishare.cc](https://apishare.cc) · **AiTun**：[aitun.cc](https://aitun.cc)
