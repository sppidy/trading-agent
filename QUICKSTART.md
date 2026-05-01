# Quickstart

Get a paper-trading stack running in ~5 minutes. This guide covers the **NSE path** (most common) end-to-end and the **Forex path** for completeness. Both run **paper-trade only** — no live order code is wired.

> [`AGENTS.md`](AGENTS.md) is the deep architectural reference. This file is the "I just want it running" guide.

---

## Prerequisites

| Path | What you need |
| --- | --- |
| **Any** | Git, `git` ≥ 2.40 (for submodules) |
| **NSE agent + backend** | Python 3.11+ |
| **Forex backend** | Docker + Docker Compose (or local PostgreSQL 14+ if you don't want Docker) |
| **Web dashboard** | Just a browser — no build step |
| **Android client** | JDK 17 + Android SDK (or use the prebuilt APK from [the v0.1.0 release](https://github.com/sppidy/janus/releases/tag/v0.1.0)) |
| **Desktop client** | .NET 8 SDK + Windows 10 19041+ (or use the prebuilt zip from the release) |

You only need the prerequisites for the components you actually run.

---

## 1. Clone with submodules

```bash
git clone --recurse-submodules https://github.com/sppidy/janus.git
cd janus
```

If you forgot `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

---

## 2. NSE path (single-user paper trading)

### 2a. Install agent + backend dependencies

```bash
# In two shells, OR in one shell with venv reuse — either works.
# NSE agent
cd nse-agent
python -m venv venv
source venv/bin/activate          # Windows: .\venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env

# NSE backend (sibling)
cd ../nse-backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
```

### 2b. Configure the agent's `.env`

Add **at least one LLM key** (any one is enough — the agent cascades through what's available):

```dotenv
# nse-agent/.env — pick whichever you have
GROQ_API_KEY=gsk_…           # console.groq.com/keys (free tier, fastest)
GEMINI_API_KEY=AIzaSy…       # aistudio.google.com/apikey
OPENROUTER_API_KEY=sk-or-…   # openrouter.ai/keys
```

Groww (`GROWW_*`) is optional — leave empty and the agent uses yfinance for OHLCV / live prices / fundamentals.

### 2c. Configure the backend's `.env`

```dotenv
# nse-backend/.env
API_AUTH_TOKEN=$(openssl rand -hex 32)   # or any long random string
TRUSTED_ORIGINS=http://localhost:8443,http://localhost:8000
AGENT_DIR=../nse-agent                    # tells backend where the agent lives
```

> **Heads-up:** the backend refuses to start with the default `change-me` token. Either set a real token or `export ALLOW_INSECURE_API_TOKEN=1` for local dev.

### 2d. Run

```bash
# Terminal 1 — backend (serves the web dashboard at /dashboard)
cd nse-backend
./start_server.sh
# Open http://localhost:8443/dashboard, paste your API_AUTH_TOKEN into the
# Settings tab. The web dashboard now talks to your backend.

# Terminal 2 (optional) — autopilot in market-hours simulation mode
cd nse-agent
source venv/bin/activate
python main.py autopilot --force
```

Try `python main.py help` for all CLI commands (`scan`, `ai-scan`, `backtest`, `train`, `chat`, etc.).

---

## 3. Forex path (multi-user, PostgreSQL, Docker Compose)

The forex backend is designed for multi-user deployments and ships as a Docker stack with PostgreSQL + the API + an autopilot worker.

```bash
# from the super (janus/) root:
cp .env.docker.example .env.docker
chmod 600 .env.docker
$EDITOR .env.docker          # set POSTGRES_PASSWORD and at least one LLM key

docker compose --env-file .env.docker up -d

# wait for the admin to be created on first boot, then pull the key:
docker compose exec forex-api cat /run/forex/admin.key
```

Now make a regular user from another shell:

```bash
ADMIN_KEY=$(docker compose exec forex-api cat /run/forex/admin.key)
curl -X POST http://localhost:8445/api/admin/users \
  -H "X-API-Key: $ADMIN_KEY" \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","display_name":"Alice"}'
# → {"username":"alice","display_name":"Alice","api_key":"fx-…"}
```

That `fx-…` key is what Alice puts into a desktop / Android / web client.

## 3a. Server install — both stacks via `install.sh`

`./install.sh` from the super root automates the full server bring-up. Run as root.

```bash
sudo ./install.sh                # NSE only (systemd units + venvs + EnvironmentFile)
sudo ./install.sh --with-forex   # NSE systemd + Forex Docker Compose
sudo ./install.sh --forex-only   # Forex Docker only (no systemd)
```

The script:
- Creates per-component venvs at `<repo>/nse-{agent,backend}/venv`.
- Seeds `/etc/janus/nse.env` (mode 0600, owner `root:ubuntu`) on first run; never overwrites it.
- Installs systemd units `janus-nse-api.service` + `janus-nse-agent.service` with hardening (`NoNewPrivileges`, `ProtectSystem=strict`, `ReadWritePaths` scoped to the repo).
- For forex: builds + brings up the Compose stack via the `.env.docker` you've filled in.

Edit `/etc/janus/nse.env` to set `API_AUTH_TOKEN` and your LLM key(s), then `sudo systemctl start janus-nse-api janus-nse-agent`.

---

## 4. Clients

You have three options for connecting to your running backend(s):

### Web dashboard (zero install)
Already served by `nse-backend` at `http://localhost:8443/dashboard`. Settings tab → paste base URL + API key.

### Desktop (Windows)
Download `Janus.Desktop-v0.1.0-x64.zip` from the [v0.1.0 release](https://github.com/sppidy/janus/releases/tag/v0.1.0), unzip, run `Janus.Desktop.exe`.

To build from source:
```bash
cd windows
msbuild Janus.Desktop.sln -p:Configuration=Debug -p:Platform=x64
# .\Janus.Desktop\bin\x64\Debug\net8.0-windows10.0.19041.0\Janus.Desktop.exe
```

If your backend uses a self-signed cert (typical for self-hosted), pin the thumbprint:
```bash
# On the backend host:
openssl x509 -in /etc/ssl/certs/forex-origin.crt -noout -fingerprint -sha1 \
  | cut -d= -f2 | tr -d :

# On the client machine:
setx TRADER_BACKEND_CERT_THUMBPRINT "<the thumbprint>"
```

### Android
Side-load `Janus-v0.1.0-release.apk` from the release, or build:
```bash
cd android
./gradlew assembleDebug
# app/build/outputs/apk/debug/app-debug.apk → adb install -r
```
Settings tab → enter your backend URL + API key.

---

## 5. Production hardening checklist

The defaults are friendly for local dev but loose for production. Set these before exposing anything beyond localhost:

| Setting | Where | Why |
| --- | --- | --- |
| Strong `API_AUTH_TOKEN` (or `ADMIN_API_KEY`) | nse-backend / forex-backend `.env` | Don't leave default values; reject all `change-me` clients. |
| `TRUSTED_ORIGINS` to your real frontend hostname | Both backends | CORS scoping; otherwise any browser-loaded site can call your API with the user's key. |
| `MODEL_HMAC_SECRET` (long random) | nse-agent + forex-agent `.env` | Pickle-load tamper resistance. After setting, re-run `python main.py train` so each model gets an `.hmac` sidecar. |
| `TRADER_BACKEND_CERT_THUMBPRINT` | Desktop client host | TLS cert pinning for self-signed backend certs. |
| Move admin API key out of stdout | forex-backend deployment | Already done — written to `/run/forex/admin.key` (mode 0600). Make sure the `forex-api` container has a writable `/run/forex` mount. |
| Front Cloudflare / nginx with TLS | Reverse proxy | Don't expose `:8443` / `:8445` directly. Origin cert can be self-signed; Cloudflare in Full mode terminates real TLS. |
| iptables: persist your firewall rules | Server host | `sudo netfilter-persistent save` after editing `/etc/iptables/rules.v4`. The 523 incident in [`docs/DEPLOY_SECURITY_NOTES.md`](docs/DEPLOY_SECURITY_NOTES.md) was caused by skipping this. |
| `CapDrop=ALL` + `NoNewPrivileges=true` on systemd units | Server host | Standard hardening for the autopilot service. |

---

## 6. Common workflows

### Update everything

```bash
git submodule foreach 'git checkout main && git pull --ff-only'
git add -A && git commit -m "Bump submodules"   # in the super
```

### Run tests for a single submodule

```bash
cd nse-backend && python -m unittest discover -s tests -v
cd nse-agent && pytest -v
```

### Add a new dependency to an agent

```bash
cd nse-agent
echo "rich>=13.7" >> requirements.txt
pip install -r requirements.txt
git add requirements.txt && git commit -S -m "deps: add rich"
git push origin main
# Bump in super:
cd .. && git add nse-agent && git commit -S -m "Bump nse-agent submodule" && git push
```

### Re-train the NSE model with HMAC sidecar

```bash
cd nse-agent
python -c "import secrets; print('MODEL_HMAC_SECRET=' + secrets.token_hex(32))" >> .env
python main.py train
# Now models/predictor_catboost.pkl.hmac exists; the agent will require it on load.
```

---

## 7. Troubleshooting

| Symptom | Probable cause | Fix |
| --- | --- | --- |
| Backend exits immediately with `RuntimeError: API_AUTH_TOKEN is unset…` | Default token in `.env` | Set a real token, OR `export ALLOW_INSECURE_API_TOKEN=1` for dev |
| Web dashboard shows `403 Invalid API key` | Browser sent wrong / no token | Settings tab → paste the same value as `API_AUTH_TOKEN` on the backend |
| Desktop client `RemoteCertificateValidationCallback returned false` | TLS cert not trusted by the new pinning policy | Either set `TRADER_BACKEND_CERT_THUMBPRINT` or `TRADER_ALLOW_INSECURE_TLS=1` |
| Predictor logs `MODEL_HMAC_SECRET is set but .hmac is missing` | Switched on HMAC after a previous train | Re-run `python main.py train` to regenerate the sidecar |
| `WS auth via ?token= is deprecated` warnings in backend logs | Using an old client | Update the client; new clients send the key as a header or first message |
| forex-api container can't write `/run/forex/admin.key` | Read-only `/run` mount | Set `ADMIN_KEY_FILE=./admin.key` in `.env`, or add a writable volume |
| Cloudflare 523 with no entries in nginx access log | iptables rules not persisted, reboot reset them | `sudo netfilter-persistent save` and re-add the 443 ACCEPT rule |

---

## 8. Where to go next

- [`AGENTS.md`](AGENTS.md) — full architectural reference, every endpoint, every CI workflow.
- [`docs/GROWW_SDK.md`](docs/GROWW_SDK.md) — Groww REST + MCP, rate limits.
- [`docs/HDFC_SKY_API.md`](docs/HDFC_SKY_API.md) — HDFC Sky endpoint reference (offline mirror).
- [`docs/DEPLOY_SECURITY_NOTES.md`](docs/DEPLOY_SECURITY_NOTES.md) — server hardening checklist (auth, TLS, sudoers).
- [`SECURITY.md`](SECURITY.md) — how to report a vulnerability privately.
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — PR workflow, sign your commits, no AI co-author trailers.
