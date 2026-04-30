# trading-agent

Mono-meta repo. Each subfolder is a git submodule pointing to its own private repo.

| Path | Repo | Purpose |
| --- | --- | --- |
| `nse-agent/` | [sppidy/nse-agent](https://github.com/sppidy/nse-agent) | NSE trading agent (Python autopilot, ML strategies) |
| `nse-backend/` | [sppidy/nse-backend](https://github.com/sppidy/nse-backend) | FastAPI backend exposing the NSE agent over WebSocket + REST |
| `android/` | [sppidy/aitrader-android](https://github.com/sppidy/aitrader-android) | Android client (Jetpack Compose, Kotlin) |
| `windows/` | [sppidy/aitrader-desktop](https://github.com/sppidy/aitrader-desktop) | Windows desktop client (.NET WinUI 3) |
| `web/` | [sppidy/aitrader-web](https://github.com/sppidy/aitrader-web) | Web dashboard (vanilla JS, lightweight-charts) |
| `forex-agent/` | [sppidy/forex-agent](https://github.com/sppidy/forex-agent) | Forex trading agent |
| `forex-backend/` | [sppidy/forex-backend](https://github.com/sppidy/forex-backend) | FastAPI backend for the forex agent |

`AGENTS.md` is the cross-repo architecture / deployment doc. `CLAUDE.md` carries shared coding-agent guidelines.

## Clone

```bash
git clone --recurse-submodules git@github.com:sppidy/trading-agent.git
```

Or after a plain clone:

```bash
git submodule update --init --recursive
```

## Update all submodules to latest main

```bash
git submodule foreach 'git checkout main && git pull --ff-only'
```
