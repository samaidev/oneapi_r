# OneAPI Gateway — Public Release Repository

This repository hosts **release packages only**. The full source code lives in the
private repository [samaidev/oneapi](https://github.com/samaidev/oneapi).

## Latest Release

👉 [Download v1.0.0 here](https://github.com/samaidev/oneapi_r/releases/latest)

## What's inside each release

| File | Platform | Includes |
|------|----------|----------|
| `oneapi-1.0.0-windows.zip` | Windows | `start.bat`, `app.py`, `tray_manager.py`, `sell_token_panel.py`, deps lists |
| `oneapi-1.0.0-linux.tar.gz` | Linux / macOS | `start.sh`, all Python sources, deps lists |
| `oneapi-1.0.0-source.zip` | Cross-platform source | Same content as above, for code review / custom builds |

## Quick start

Unzip → run `start.bat` (Windows) or `./start.sh` (Linux/macOS). OneAPI Gateway
starts on `http://localhost:3000`. Right-click the tray icon → **Sell token...**
to publish your `mymodel` to [apishare.cc](https://apishare.cc).

Full documentation in [README.md](./README.md).
