# OneAPI Gateway

> **English** | **[中文](./README.zh-CN.md)**

OneAPI Gateway is a high-performance aggregated LLM API gateway. It combines OpenAI, Anthropic, Gemini, DeepSeek, Azure and other upstream providers behind one unified OpenAI-compatible endpoint — a single static binary with a system tray manager and built-in admin dashboard. This repository publishes ready-to-run release packages only; the source lives in the development repository [samaidev/oneapi](https://github.com/samaidev/oneapi).

---

## ✨ Features

- **Single static binary** — no runtime, no dependencies; download, run, done (~17–19 MB per platform)
- **Multi-Provider Aggregation** — OpenAI, Anthropic, Gemini, DeepSeek, Azure and more behind one unified OpenAI-compatible endpoint
- **Universal Model Alias (`mymodel`)** — call any model as `mymodel`; requests are load-balanced across all active channels automatically
- **System Tray Manager** — start, stop, restart, configure, and sell spare capacity from the tray icon (Windows real systray, macOS menu-bar icon; Linux headless)
- **Built-in Admin Dashboard** — channels, tokens, logs, and configuration at `http://localhost:3000`
- **Per-Key Rate Limits** — custom API keys with per-minute RPM limits and total token quota caps
- **Daily Auto-Reset & Health Checks** — quota-exhaustion circuit-break with auto recovery at 00:00, daily channel testing
- **Free-Model Auto-Load** — on startup the embedded AI agent automatically syncs the latest free models from opencode.ai, so it works out of the box with no API keys
- **Path Compatibility** — supports both `/v1/chat/completions` and `/chat/completions` (drop-in replacement for the OpenAI SDK)
- **AiTun Integration** — paste your `ait_xxx` secret key to expose a public domain automatically
- **APIShare Sell Token** — one-click publish your `mymodel` on apishare.cc and earn from spare capacity

## 📥 Download

Grab the latest release from the **[Releases page](https://github.com/samaidev/oneapi_r/releases/latest)** (current: **v2.0.0**):

| File | Platform | Size |
|---|---|---|
| [`oneapi-windows-amd64.exe`](https://github.com/samaidev/oneapi_r/releases/download/v2.0.0/oneapi-windows-amd64.exe) | Windows x86_64 | ~18 MB |
| [`OneAPI-v2.0.0.dmg`](https://github.com/samaidev/oneapi_r/releases/download/v2.0.0/OneAPI-v2.0.0.dmg) | macOS (OneAPI.app) | ~12 MB |
| [`OneAPI-v2.0.0-macOS.zip`](https://github.com/samaidev/oneapi_r/releases/download/v2.0.0/OneAPI-v2.0.0-macOS.zip) | macOS (zipped app) | ~11 MB |
| [`oneapi-linux-amd64`](https://github.com/samaidev/oneapi_r/releases/download/v2.0.0/oneapi-linux-amd64) | Linux x86_64 | ~18 MB |
| [`oneapi-darwin-amd64`](https://github.com/samaidev/oneapi_r/releases/download/v2.0.0/oneapi-darwin-amd64) | macOS raw binary | ~19 MB |

## 🚀 Install & Run

### Windows

1. Download `oneapi-windows-amd64.exe` (button above or from the [Releases page](https://github.com/samaidev/oneapi_r/releases/latest)).
2. Double-click to run — no installation needed.
3. Open http://localhost:3000 in your browser. Default password: `admin123`.

### macOS & Linux (one-line installer — recommended)

```bash
curl -fsSL https://apishare.cc/oneapi-install.sh | bash
```

The installer downloads the right package, installs it, sets up auto-start at login, and launches OneAPI in the background:

- **macOS**: installs to `/Applications/OneAPI.app` + LaunchAgent (menu-bar icon appears)
- **Linux**: installs to `~/.local/share/oneapi` + systemd user service

Then open http://localhost:3000 in your browser. Default password: `admin123`.

### Manual install (if the one-liner doesn't work)

**Step 1 — install the app / binary**

- **macOS**: download the `.dmg`, mount it, drag `OneAPI.app` to Applications.
- **Linux**: download `oneapi-linux-amd64`, then:

```bash
mkdir -p ~/oneapi && cd ~/oneapi
chmod +x oneapi-linux-amd64
./oneapi-linux-amd64
```

**Step 2 — macOS only, remove quarantine (once)**

```bash
xattr -dr com.apple.quarantine /Applications/OneAPI.app
```

**Step 3 — launch (macOS terminal)**

```bash
mkdir -p ~/oneapi && cd ~/oneapi && /Applications/OneAPI.app/Contents/MacOS/oneapi-gateway
```

Keep the terminal window open (closing it stops OneAPI). The tray icon shows "OneAPI - Running". For auto-start at login, prefer the one-line installer above.

## 🖥️ First Run

1. Open **http://localhost:3000** and sign in (default password `admin123` — change it after first login).
2. Add **channels** (DeepSeek, Claude, OpenAI, etc.) in the dashboard.
3. Call any model through the unified endpoint with `model: "mymodel"` — requests are routed to the healthiest active channel:

```bash
curl http://localhost:3000/v1/chat/completions \
  -H "Authorization: Bearer sk-your-oneapi-token" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "mymodel",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

The embedded sidebar AI agent is ready immediately — on every startup it auto-syncs the latest free models from opencode.ai (no API key required).

## 💰 Sell Your Spare Capacity on APIShare

1. Get an APIShare agent secret key (`apsagt-...`) from [apishare.cc/dashboard](https://apishare.cc/dashboard).
2. Right-click the OneAPI tray icon → **Sell token...** (or open `http://localhost:3000/sell-token`).
3. Paste the agent secret key and click **OK & Start Selling** — OneAPI launches the tunnel, publishes your `mymodel` on apishare.cc, and starts routing paid traffic. You earn 90% of every call. To stop, click **Stop selling** on the same page.

## ❓ Troubleshooting

- **Can't log in** — default admin password is `admin123`; change it after first login.
- **No tray icon on Linux/macOS** — Linux runs headless by design (open the dashboard in a browser); on macOS the menu-bar icon needs the app installed under `/Applications`.
- **macOS blocks the app ("unidentified developer")** — run `xattr -dr com.apple.quarantine /Applications/OneAPI.app` once.
- **Port 3000 already in use** — set `PORT=<other>` in the `.env` file next to the binary (or the Config Panel at `http://localhost:3000/config`) and restart.

## 🔗 Links

- **Install guide & docs**: https://apishare.cc/oneapi
- **Source (development repo)**: [samaidev/oneapi](https://github.com/samaidev/oneapi)
- **Releases**: [samaidev/oneapi_r/releases/latest](https://github.com/samaidev/oneapi_r/releases/latest)
- **APIShare**: [apishare.cc](https://apishare.cc) · **AiTun**: [aitun.cc](https://aitun.cc)
