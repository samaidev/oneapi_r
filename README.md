# OneAPI Gateway (Go Edition)

> **English** | **[中文](./README.zh-CN.md)**

High-performance aggregated LLM API gateway, rewritten in Go from the original Python version. Combines multiple upstream providers (OpenAI, Anthropic, Gemini, DeepSeek, Azure, etc.) behind one unified OpenAI-compatible endpoint, with system tray + AiTun + APIShare tunnel integration.

**Version**: 2.0.0
**Language**: Go 1.23+
**License**: MIT

---

## ✨ Why Go?

The original Python version ([backed up at ctz168/oneapi_py](https://github.com/ctz168/oneapi_py)) worked well but had one pain point: **PyInstaller cannot cross-compile**. Producing Windows `.exe`, macOS binary, and Linux binary required either building on each platform manually or maintaining complex GitHub Actions matrix builds.

Go solves this elegantly — a single `go build` command with `GOOS=windows GOARCH=amd64` produces a Windows binary from Linux, with zero CGo dependencies. All three platforms build in seconds on any machine.

## ✨ Features

- **Single static binary** — no Python install needed, no shared lib dependencies, ~17 MB per platform
- **True cross-platform build** — `GOOS=windows GOARCH=amd64 go build` works from Linux (and vice versa)
- **System Tray** — Windows uses `getlantern/systray` (real tray icon). Linux/macOS run headless (HTTP server only).
- **Universal Model Alias (`mymodel`)** — auto load-balanced routing across all active channels
- **Multi-Provider Aggregation** — OpenAI, Anthropic, Gemini, DeepSeek, Azure behind one endpoint
- **Per-Key Rate Limits** — RPM + total token quotas per API key
- **Daily Auto-Reset** — quota-exhaustion circuit-break + auto recovery at 00:00
- **Path Compatibility** — both `/v1/chat/completions` and `/chat/completions`
- **AiTun Integration** — paste your `ait_xxx` secret key, auto-launches tunnel + discovers public domain
- **APIShare Sell Token** — one-click publish `mymodel` to apishare.cc marketplace
- **Database Compatible** — reads existing Python-created `oneapi.db` directly (same SQLite schema)

## 🚀 Quick Start

### Option 1: Download a release

Go to [latest release page](https://github.com/samaidev/oneapi_r/releases/latest) and download the binary for your platform:
- `oneapi-linux-amd64` — Linux x86_64 (~17 MB)
- `oneapi-windows-amd64.exe` — Windows x86_64 (~18 MB)
- `oneapi-darwin-arm64` — macOS Apple Silicon (~16 MB)

Run:
```bash
chmod +x oneapi-linux-amd64
./oneapi-linux-amd64    # starts on http://localhost:3000
```

### Option 2: Build from source

```bash
git clone https://github.com/samaidev/oneapi.git
cd oneapi
go build -o oneapi-gateway ./cmd/oneapi-gateway
./oneapi-gateway
```

### Option 3: Cross-compile for all 3 platforms

```bash
# From any platform — produces all 3 binaries
GOOS=linux GOARCH=amd64 go build -o dist/oneapi-linux-amd64 ./cmd/oneapi-gateway
GOOS=windows GOARCH=amd64 go build -o dist/oneapi-windows-amd64.exe ./cmd/oneapi-gateway
GOOS=darwin GOARCH=arm64 CGO_ENABLED=0 go build -o dist/oneapi-darwin-arm64 ./cmd/oneapi-gateway
```

That's it. No Docker, no PyInstaller spec, no GitHub Actions matrix, no toolchain hassles.

## 🛠️ Configuration

Edit `.env`:

```env
PORT=3000
DATABASE_URL=sqlite+aiosqlite:///./oneapi.db
AITUN_SECRET_KEY=    # optional: ait_xxx from https://aitun.cc/ for public domain
```

Or use the **Config Panel** at `http://localhost:3000/config` (browser UI).

### AiTun Public Domain

Sign in at [aitun.cc](https://aitun.cc/), register a subdomain, copy your `ait_xxx` secret key, paste into `.env` or the Config Panel. OneAPI auto-launches `aitun -k <key> -p <port>` and discovers the public domain (~5s).

### Channels (Upstream Providers)

1. Open `http://localhost:3000` (default admin password: `admin123`)
2. Add channels (DeepSeek, Claude, OpenAI, etc.)
3. Configure model mapping — `model: "mymodel"` auto-routes to any active channel

## 💰 Sell Token to APIShare

1. Get an APIShare agent secret key (`apsagt-...`) from [apishare.cc/dashboard](https://apishare.cc/dashboard)
2. Open `http://localhost:3000/sell-token` in browser
3. Paste the key + click "OK & Start Selling"
4. OneAPI auto-creates local token + registers tunnel provider + launches `apishare-tunnel` + activates

To stop: click "Stop selling" on the same page.

## 📖 API Usage

OneAPI is a drop-in OpenAI replacement:

```bash
curl http://localhost:3000/v1/chat/completions \
  -H "Authorization: Bearer sk-your-oneapi-token" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "mymodel",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

## 🧱 Architecture

```
Your apps / OpenAI SDK
    │  Bearer sk-oneapi-...
    ▼
┌───────────────────────────────────────┐
│  OneAPI Gateway (Go binary)          │
│  - chi router + CORS                 │
│  - balancer (mymodel routing)         │
│  - proxy (upstream forwarding + SSE)  │
│  - rate_limiter + adaptive_limiter    │
│  - reasoning_normalizer              │
│  - i18n (embed JSON, en + zh)         │
│  - tray (Windows systray)            │
│  - aitun (tunnel auto-start)          │
│  - apishare (sell token flow)         │
└───────────────────────────────────────┘
    │  model=mymodel (load-balanced)
    ▼
  OpenAI / Claude / Gemini / DeepSeek / Azure

Optional public access:
  ┌───────────────────────────────────────┐
  │  aitun.cc tunnel (auto-launched)      │
  │  → myapp.t.aitun.cc                   │
  └───────────────────────────────────────┘
```

## 📁 Project Structure

```
oneapi/
├── cmd/oneapi-gateway/main.go    # Entry point
├── internal/
│   ├── config/                   # .env + env var loading
│   ├── db/                       # sqlx + SQLite (modernc.org/sqlite, no CGo)
│   ├── models/                   # Channel, ApiToken, RequestLog, etc.
│   ├── balancer/                 # mymodel routing + midnight reset
│   ├── limiter/                  # RateLimiter + AdaptiveLimiter
│   ├── proxy/                    # Upstream forwarding + reasoning normalizer + utils
│   ├── handlers/                 # OpenAI + admin API (25+ endpoints)
│   ├── tray/                     # System tray (Windows only)
│   ├── aitun/                    # AiTun tunnel auto-launch
│   ├── apishare/                 # APIShare sell-token flow + web UI
│   └── i18n/                     # Embed JSON translations (en + zh)
├── web/
│   ├── static/                   # Frontend HTML/JS/CSS (reused from Python version)
│   └── web.go                    # embed.FS for static assets
├── .env.example
├── go.mod / go.sum
└── README.md
```

## 🔧 Troubleshooting

- **Linux/macOS: no tray icon** — by design. Linux requires `libayatana-appindicator3-dev` (build with `-tags=appindicator`), macOS requires Cocoa/CGo. Out of the box we run headless on these platforms — open `http://localhost:3000` in browser instead.
- **Database error "out of memory"** — ensure `.env` has `DATABASE_URL=sqlite+aiosqlite:///./oneapi.db` (or absolute path).
- **Can't login** — default admin password is `admin123`. Change it after first login.
- **AiTun not starting** — ensure `aitun-client-*` binary is on PATH or next to OneAPI binary. Download from [aitun.cc/downloads](https://aitun.cc/downloads).

## 🔗 Links

- **Source**: [samaidev/oneapi](https://github.com/samaidev/oneapi)
- **Public releases**: [samaidev/oneapi_r](https://github.com/samaidev/oneapi_r/releases/latest)
- **Python backup**: [ctz168/oneapi_py](https://github.com/ctz168/oneapi_py)
- **APIShare**: [apishare.cc](https://apishare.cc)
- **AiTun**: [aitun.cc](https://aitun.cc)
