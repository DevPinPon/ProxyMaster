# ProxyMaster
One-click IPv6 proxy creation using your home network. Generate unlimited residential IPv6 proxies from your ISP-assigned /64 block and manage isolated Chrome browser profiles — each with its own identity, cookies, and proxy.

![Windows](https://img.shields.io/badge/platform-Windows-blue) ![Electron](https://img.shields.io/badge/electron-33-47848F) ![License](https://img.shields.io/badge/license-MIT-green)

## How It Works

Your ISP assigns you an IPv6 /64 block (~18 quintillion addresses). ProxyMaster binds random addresses from that block to your network interface, then runs [3proxy](https://github.com/3proxy/3proxy) in WSL to serve HTTP/SOCKS5 proxies — each with a unique IPv6 exit IP.

```
Windows (Electron App)              WSL (Ubuntu)
┌──────────────────────┐           ┌─────────────────────┐
│  React UI            │           │  3proxy (HTTP/SOCKS) │
│  ├─ Dashboard        │  ←IPC→   │  ndppd (NDP proxy)   │
│  ├─ Proxy Manager    │           │  IPv6 routing        │
│  ├─ Chrome Profiles  │           └─────────────────────┘
│  └─ Settings         │
│                      │           Wi-Fi / Ethernet
│  IPv6 binding ───────┼──→ netsh interface ipv6 add address
│  Chrome launcher ────┼──→ chrome.exe --proxy-server=...
└──────────────────────┘
```

## Features

- **One-click proxy creation** — select count & protocol, proxies are live in seconds
- **Unique IPv6 per proxy** — each proxy exits through a different residential IPv6
- **HTTP + SOCKS5** — both protocols supported, individually or combined
- **Chrome browser profiles** — isolated instances with own cookies, cache, history, and assigned proxy
- **WebRTC leak protection** — Chrome profiles block WebRTC IP leaks automatically
- **Auto-restore on startup** — proxies rebind automatically when the app opens
- **Export formats** — `ip:port:user:pass`, `user:pass@ip:port`, JSON, CSV
- **No bot detection** — real Chrome browser, not automated (no Selenium/Puppeteer flags)

## Requirements

- **Windows 10/11** with WSL enabled
- **WSL Ubuntu** (22.04 recommended) — `wsl --install -d Ubuntu-22.04`
- **IPv6 connectivity** — your ISP must provide a /64 or larger IPv6 block
- **Google Chrome** installed

## Quick Start

```bash
# Clone and install
git clone https://github.com/your-username/ProxyMaster.git
cd ProxyMaster
npm install

# Development
npm run dev

# Build portable .exe
npm run build:portable
```

### First Run

1. Open the app → go to **Settings**
2. Select your WSL distro (e.g. `Ubuntu-22.04`)
3. Click **Install Dependencies** (installs 3proxy + ndppd in WSL)
4. Click **Detect IPv6** — the app finds your prefix automatically
5. Go to **Proxies** → **Create** → pick count & protocol → done

### Using Chrome Profiles

1. Go to **Profiles** → **New Profile**
2. Name it, pick a color, assign a running proxy
3. Click **Launch Chrome** — opens an isolated Chrome instance
4. Each profile has its own cookies, history, and exits through its assigned IPv6

## Architecture

| Layer | Tech | Purpose |
|-------|------|---------|
| UI | React + TypeScript + Tailwind CSS | Frontend renderer |
| State | Zustand | Proxy, settings, profile stores |
| Desktop | Electron + electron-vite | Window management, IPC |
| Proxy server | 3proxy (in WSL) | HTTP/SOCKS5 proxy with IPv6 binding |
| NDP | ndppd (in WSL) | Advertises IPv6 addresses to router |
| IPv6 binding | netsh (Windows) | Binds /128 addresses to interface |
| Browser | Chrome | Isolated profiles via `--user-data-dir` |

## Project Structure

```
ProxyMaster/
├── electron/
│   ├── main.ts                 # App entry, frameless window
│   ├── preload.ts              # Context bridge API
│   └── services/
│       ├── ipc-handlers.ts     # All IPC orchestration
│       ├── wsl-executor.ts     # Run commands in WSL
│       ├── ipv6-manager.ts     # Detect, generate, bind IPv6
│       ├── proxy-engine.ts     # 3proxy config & lifecycle
│       ├── ndppd-manager.ts    # ndppd config & lifecycle
│       ├── proxy-store.ts      # JSON persistence
│       └── chrome-manager.ts   # Chrome profile launcher
├── src/
│   ├── App.tsx
│   ├── pages/                  # Dashboard, Proxies, Profiles, Settings, Logs
│   ├── components/             # UI components (glass cards, buttons, modals)
│   ├── stores/                 # Zustand stores
│   ├── hooks/                  # useWSLStatus, useProxies, etc.
│   └── lib/                    # IPC wrappers, formatters
├── scripts/
│   ├── dev.js                  # Dev wrapper (fixes ELECTRON_RUN_AS_NODE)
│   └── wsl/                    # Shell scripts for WSL setup
└── package.json
```

## Proxy Export Formats

```
# ip:port:user:pass
2600:4040:5734:8f00:3bbb:5bf0:707c:b531:20000:pinpon:032785

# user:pass@ip:port
pinpon:032785@[2600:4040:5734:8f00:3bbb:5bf0:707c:b531]:20000

# Test with curl
curl -x http://user:pass@127.0.0.1:20000 https://api64.ipify.org
```

## Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Start in development mode with HMR |
| `npm run build` | Build for production |
| `npm run build:win` | Build Windows installer (.exe) |
| `npm run build:portable` | Build portable executable |
| `npm run build:unpack` | Build unpacked directory |

## Tech Stack

React 18 · TypeScript · Tailwind CSS · Zustand · Framer Motion · Electron 33 · electron-vite · 3proxy · ndppd

## License

MIT
