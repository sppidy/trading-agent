# trading-agent

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![CI](https://github.com/sppidy/trading-agent/actions/workflows/ci.yml/badge.svg)](https://github.com/sppidy/trading-agent/actions/workflows/ci.yml)

Open-source paper-trading platform spanning **Indian equities (NSE)** and **Forex**, with a shared HTTP + WebSocket contract powering web, desktop, and Android clients.

This is a **meta-repo**: every component is its own git submodule with its own license, CI, and issue tracker.

> ⚠️ **Disclaimer:** Paper-trading only — no live order code is wired. Not financial advice. Trading involves risk.

## Layout

| Path | Repo | Stack | Role |
| --- | --- | --- | --- |
| [`nse-agent/`](https://github.com/sppidy/nse-agent) | sppidy/nse-agent | Python, CatBoost | NSE trading agent (rule-based + LLM + ML) |
| [`nse-backend/`](https://github.com/sppidy/nse-backend) | sppidy/nse-backend | FastAPI, uvicorn | Single-user backend wrapping `nse-agent` |
| [`web/`](https://github.com/sppidy/aitrader-web) | sppidy/aitrader-web | Vanilla JS | Dashboard served by `nse-backend` at `/dashboard` |
| [`windows/`](https://github.com/sppidy/aitrader-desktop) | sppidy/aitrader-desktop | WinUI 3 / .NET 8 | Desktop client |
| [`android/`](https://github.com/sppidy/aitrader-android) | sppidy/aitrader-android | Kotlin / Jetpack Compose | Mobile client |
| [`forex-agent/`](https://github.com/sppidy/forex-agent) | sppidy/forex-agent | Python | Forex agent (ICT-style strategies) |
| [`forex-backend/`](https://github.com/sppidy/forex-backend) | sppidy/forex-backend | FastAPI + PostgreSQL + Docker | Multi-user forex backend |

## Architecture

```
Web dashboard ─┐
Desktop app ───┤     ┌── NSE backend (single-user, in-memory jobs)
Android app  ──┼────►│      └── dynamically imports nse-agent modules
               │     │
               └────►└── Forex backend (multi-user, PostgreSQL, Docker)
                              └── imports forex-agent modules
```

All three clients speak the **same HTTP + WebSocket contract**. Each stores per-profile base URL + API key, so one client binary flips between NSE main, NSE eval, and Forex backends without rebuild.

Full architectural reference: [`AGENTS.md`](AGENTS.md).

## Getting started

### Clone with submodules

```bash
git clone --recurse-submodules https://github.com/sppidy/trading-agent.git
cd trading-agent
# or, if you already cloned without --recurse-submodules:
git submodule update --init --recursive
```

### Pull every submodule to its `main`

```bash
git submodule foreach 'git checkout main && git pull --ff-only'
```

### Run the NSE stack locally

```bash
# 1. NSE agent
cd nse-agent
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env                       # add at least one LLM key

# 2. NSE backend (in another shell)
cd ../nse-backend
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env                       # set API_AUTH_TOKEN
export AGENT_DIR=../nse-agent
./start_server.sh
```

Then open `http://localhost:8443/dashboard` (or your configured port) for the web UI, or build the desktop / Android client and point it at your backend.

### Run the Forex stack locally

```bash
cd forex-backend
cp .env.example .env                       # set POSTGRES_PASSWORD + ADMIN_API_KEY
docker compose up -d
```

## Documentation

- [`AGENTS.md`](AGENTS.md) — full architectural reference (every submodule, every endpoint, every CI workflow)
- [`docs/GROWW_SDK.md`](docs/GROWW_SDK.md) — Groww REST + MCP reference, rate limits
- [`docs/HDFC_SKY_API.md`](docs/HDFC_SKY_API.md) — HDFC Sky API endpoint notes (offline mirror of the dev portal)
- [`docs/DEPLOY_SECURITY_NOTES.md`](docs/DEPLOY_SECURITY_NOTES.md) — server hardening checklist for the FastAPI backends

## CI / build

Workflows live in [`.github/workflows/`](.github/workflows):

| File | What it builds |
| --- | --- |
| `ci.yml` | pytest for `nse-agent` + `nse-backend`; `compileall` smoke for forex agent/backend; gradle unit tests for `android/`; asset-presence check for `web/` |
| `android-build.yml` | Debug + signed release APK from `android/` (uses `RELEASE_*` GH secrets) |
| `desktop-build.yml` | msbuild Release x64 of the WinUI solution under `windows/` |

Super-level CI requires a `SUBMODULE_PAT` secret (fine-grained PAT with read access to all 7 submodule repos) so it can fetch private submodules. The default `GITHUB_TOKEN` cannot fetch other repos. Once the project is fully public, this requirement goes away.

## Contributing

PRs welcome. Read [`CONTRIBUTING.md`](CONTRIBUTING.md) first — short version: open an issue first for non-trivial changes, target the relevant submodule, sign your commits, and don't add `Co-Authored-By` AI trailers.

## Security

Don't open public issues for security problems. See [`SECURITY.md`](SECURITY.md).

## License

[MIT](LICENSE) — see file for the full text and the no-financial-advice disclaimer.
