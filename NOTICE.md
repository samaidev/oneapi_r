# OneAPI Gateway — Public Release Repository

This repository hosts **release packages only**. The full source code lives in the
development repository [samaidev/oneapi](https://github.com/samaidev/oneapi).

## Latest Release

👉 [Download the latest release](https://github.com/samaidev/oneapi_r/releases/latest)

## What's inside each release (v2.0.0, Go edition)

| File | Platform | Notes |
|------|----------|-------|
| `oneapi-windows-amd64.exe` | Windows x86_64 | Single executable with real systray; double-click to run |
| `OneAPI-v2.0.0.dmg` / `OneAPI-latest.dmg` | macOS | Contains `OneAPI.app` (`OneAPI.app/Contents/MacOS/oneapi-gateway`) |
| `OneAPI-v2.0.0-macOS.zip` / `OneAPI-latest.zip` | macOS | Zipped `OneAPI.app`, same content as the DMG |
| `oneapi-darwin-amd64` | macOS | Raw binary (headless / terminal launch) |
| `oneapi-linux-amd64` | Linux x86_64 | Single executable (headless, HTTP on :3000) |

`OneAPI-latest.dmg` / `OneAPI-latest.zip` are version-independent aliases: the URLs
`https://github.com/samaidev/oneapi_r/releases/latest/download/OneAPI-latest.dmg` (and `.zip`)
always point at the newest release.

Install guide: https://apishare.cc/oneapi · one-liner (macOS/Linux):
`curl -fsSL https://apishare.cc/oneapi-install.sh | bash`

After launch, open http://localhost:3000 (default password: `admin123`).
