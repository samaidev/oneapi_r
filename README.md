# OneAPI Gateway

> **[English](./README.md)** · **[中文](./README.zh-CN.md)**

OneAPI Gateway is a high-performance Python-based LLM API aggregator. It combines multiple upstream providers (OpenAI, Anthropic, Gemini, DeepSeek, Azure, and more) behind one unified OpenAI-compatible endpoint, with a Windows system tray manager and one-click [APIShare](https://apishare.cc) integration.

**Version**: 1.0.0  
**License**: MIT  
**Status**: Production-ready

---

## ✨ Features

- **System Tray Manager** — Start, stop, restart, configure, and one-click "Sell token..." to publish your spare capacity to APIShare, all from the Windows system tray.
- **Universal Model Alias (`mymodel`)** — Call any model as `mymodel` and OneAPI auto-routes to the cheapest/healthiest active channel — true load-balanced routing.
- **Multi-Provider Aggregation** — OpenAI, Anthropic, Gemini, DeepSeek, Azure, and more — all behind one unified OpenAI-compatible endpoint.
- **Daily Auto-Reset & Testing** — Smart quota-exhaustion detection with auto-circuit-break. Daily 12:05 channel health checks. Daily 00:00 quota reset.
- **Per-Key Rate Limits** — Generate custom API keys with per-minute RPM limits and total quota caps.
- **Path Compatibility** — Supports both `/v1/chat/completions` and `/chat/completions` — drop-in replacement for the OpenAI SDK.
- **One-Click APIShare Integration** — Right-click tray → "Sell token..." → paste your APIShare agent token → OneAPI automatically creates a local token, registers `mymodel` as a tunnel-mode provider on APIShare, launches the APIShare tunnel client, and activates it. You earn 90% of every call.

---

## 🚀 Quick Start

### Option 1: Download a release package

1. Go to the [latest release page](https://github.com/samaidev/oneapi_r/releases/latest).
2. Download the package for your platform:
   - `oneapi-1.0.0-windows.zip` — Windows (with `start.bat`)
   - `oneapi-1.0.0-linux.tar.gz` — Linux/macOS (with `start.sh`)
   - `oneapi-1.0.0-source.zip` — Cross-platform source
3. Unzip and run:
   - **Windows**: double-click `start.bat`
   - **Linux/macOS**: `./start.sh`
4. The tray icon appears. Service starts on `http://localhost:3000`.

### Option 2: From source

```bash
git clone https://github.com/samaidev/oneapi.git
cd oneapi
pip install -r requirements.txt -r tray-requirements.txt
python app.py        # headless mode
# OR
python tray_manager.py   # with tray (Windows recommended)
```

---

## 🛠️ Configuration

### Service Port

Edit `.env`:

```env
PORT=3000
DATABASE_URL=sqlite+aiosqlite:///./oneapi.db
DUCKDNS_DOMAIN=
DUCKDNS_TOKEN=
```

Apply by right-clicking tray → **Restart service**.

### Channels (Upstream Providers)

1. Right-click tray → **Open Dashboard** (or visit `http://localhost:3000`).
2. Default admin password: `admin123` (change it after first login).
3. Add channels (DeepSeek, Claude, OpenAI, Gemini, etc.).
4. Configure model mapping:
   - Format: `alias:real_model` (e.g. `pro-chat:gpt-4o`)
   - **Universal mode**: requests with `model: "mymodel"` are auto-routed to any active channel and use its first model. This is what APIShare will publish.

### API Tokens

In the dashboard → **Tokens** tab:
- Create per-client tokens with `max_rpm` and `max_tokens` limits.
- The "Sell token..." feature automatically creates a token named `apishare-cc-public` for APIShare's public traffic.

---

## 💰 Sell Token to APIShare (One-Click)

The fastest way to monetize your local `mymodel`:

1. **Get an APIShare agent token**: log in to [apishare.cc/dashboard](https://apishare.cc/dashboard) → Agent Tokens card → create a token with `provider:write` permission (or `*` for all). Copy the `apsagt-...` value.
2. **Open the dialog**: right-click the OneAPI tray icon → **Sell token...**.
3. **Paste the agent token** in the input box. Adjust the local port (default 3000) and model name (default `mymodel`) if needed.
4. **Click "OK & Start Selling"**. OneAPI will:
   - Validate the agent token against `https://apishare.cc/api/user/agent-tokens`.
   - Locally create a OneAPI token named `apishare-cc-public` (this is what APIShare's public callers will use).
   - Register a tunnel-mode provider on APIShare with `model_name=mymodel` and `api_key=<local_token>`.
   - Install `apishare-tunnel` (via pip) if missing, then launch `apishare -k <tunnelToken> -p 3000` as a background subprocess.
   - Wait for the tunnel to connect, then call `POST /api/provider/activate/{api_id}` to activate.
5. **Done.** Your `mymodel` is now publicly callable through `https://apishare.cc/v1/chat/completions` (or via the APIShare market page). You earn 90% of every call.

> 💡 **Hyperlink in dialog**: the input area has a clickable "Open apishare.cc/dashboard" link if you need to grab your agent token mid-flow.

### How the auth handshake works

- APIShare consumers send `Authorization: Bearer <apishare_consumer_key>` to apishare.cc.
- APIShare validates the consumer key, deducts balance, then forwards the request through the standalone tunnel server (port 3397) to your local apishare-tunnel client.
- The apishare platform injects `Authorization: Bearer <your_local_oneapi_token>` (stored when registering the provider) so your local OneAPI can authenticate the forwarded request.
- OneAPI then routes the request to `mymodel` — i.e. load-balanced across all your active upstream channels.

---

## 📖 API Usage

Once OneAPI is running on `http://localhost:3000`, it's a drop-in OpenAI replacement:

```bash
curl http://localhost:3000/v1/chat/completions \
  -H "Authorization: Bearer sk-your-oneapi-token" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "mymodel",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

Both `/v1/chat/completions` and `/chat/completions` work.

---

## 🧱 Architecture

```
                          ┌──────────────────────────────┐
                          │  Your apps / OpenAI SDK       │
                          └────────────┬─────────────────┘
                                       │ Bearer sk-oneapi-...
                                       ▼
                          ┌──────────────────────────────┐
                          │  OneAPI Gateway (Python)      │
                          │  - aiohttp + SQLAlchemy       │
                          │  - pystray (Windows tray)     │
                          │  - sell_token_panel.py        │
                          │    └─ creates local token     │
                          │    └─ registers on APIShare   │
                          │    └─ launches apishare-tunnel│
                          └────────────┬─────────────────┘
                                       │ model=mymodel (load-balanced)
                                       ▼
                  ┌─────────────┬──────┴──────┬─────────────┐
                  ▼             ▼             ▼             ▼
            ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
            │ OpenAI  │  │Claude   │  │ Gemini  │  │ DeepSeek│
            └─────────┘  └─────────┘  └─────────┘  └─────────┘

                          ┌──────────────────────────────┐
                          │  apishare.cc (Go platform)    │
                          │  - Consumer requests          │
                          │  - Forwards via tunnel (3397)  │
                          └────────────┬─────────────────┘
                                       │ WSS tunnel + Bearer <local_token>
                                       ▼
                          ┌──────────────────────────────┐
                          │  apishare-tunnel client       │
                          │  (Python, launched by tray)   │
                          └────────────┬─────────────────┘
                                       │ HTTP localhost:3000
                                       ▼
                          ┌──────────────────────────────┐
                          │  OneAPI Gateway               │
                          └──────────────────────────────┘
```

---

## 📁 Project Structure

```
oneapi/
├── app.py                  # Main aiohttp server (port 3000)
├── tray_manager.py         # Windows system tray manager
├── sell_token_panel.py    # APIShare integration dialog (NEW)
├── config_panel.py        # .env editor dialog (Tkinter)
├── i18n.py                # English (default) + 中文 translations
├── admin.py               # Admin API (login, channels, tokens, logs)
├── handlers.py            # /v1/chat/completions, /v1/models
├── core/
│   ├── load_balancer.py   # mymodel routing logic
│   ├── proxy.py           # Upstream provider proxying
│   ├── database.py        # SQLAlchemy models
│   └── ...
├── static/                # Dashboard UI
├── requirements.txt
├── tray-requirements.txt
├── start.bat              # Windows launcher
├── start.sh               # Linux/macOS launcher
└── launcher-select.bat   # Pick launcher mode
```

---

## 🔧 Troubleshooting

- **Tray icon missing on Linux**: install `python3-tk` + `libappindicator3-dev` and re-run `pip install pystray`.
- **"Service started but not detected"**: check `service.log` for upstream errors.
- **"apishare-tunnel: command not found"**: the Sell token dialog auto-installs it via pip; if it still fails, run `pip install apishare-tunnel` manually.
- **"Tunnel not connected" warning on activate**: the tunnel may need more than 8 seconds to register. Wait 30s and click **Sell token...** again — OneAPI will detect the existing state and retry activation. You can also activate manually from the apishare.cc dashboard.
- **Local admin login fails**: if you changed the default password from `admin123`, the Sell token dialog cannot auto-create a local token. Either reset to default, or manually create a token named `apishare-cc-public` in the dashboard and use that.

---

## 📜 License

MIT. See [LICENSE](./LICENSE).

## 🙏 Acknowledgements

- [APIShare](https://apishare.cc) — for the tunnel service + marketplace
- [aiohttp](https://docs.aiohttp.org/) — async HTTP server
- [pystray](https://github.com/moses-palmer/pystray) — cross-platform tray
- [Pillow](https://python-pillow.org/) — icon rendering

## 🔗 Links

- **Private source repo**: [samaidev/oneapi](https://github.com/samaidev/oneapi)
- **Public release repo**: [samaidev/oneapi_r](https://github.com/samaidev/oneapi_r/releases/latest)
- **APIShare website**: [apishare.cc](https://apishare.cc) · [/oneapi](https://apishare.cc/oneapi) intro page
