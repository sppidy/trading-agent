# Security Policy

## Reporting a vulnerability

**Do not open a public GitHub issue for security problems.**

Use GitHub's [private vulnerability reporting](https://github.com/sppidy/janus/security/advisories/new) on the affected repo, or email the maintainer directly. Include:

- A clear description of the issue
- Steps to reproduce (proof-of-concept welcome)
- The affected submodule(s) and version / commit SHA
- Your assessment of the impact

You'll get an acknowledgement within 5 business days.

## Scope

This policy covers the seven submodules of [`janus`](https://github.com/sppidy/janus):

`nse-agent`, `nse-backend`, `aitrader-android`, `aitrader-desktop`, `aitrader-web`, `forex-agent`, `forex-backend`.

Out of scope:

- Vulnerabilities in upstream dependencies (please report those upstream; we'll bump versions when fixes land)
- Issues that require physical access to a user's machine
- Social-engineering / phishing of project maintainers

## What we care about most

- Auth bypass on the FastAPI backends (`nse-backend`, `forex-backend`)
- Token / credential leakage in logs, error responses, or git history
- SQL injection / unsafe query building
- WebSocket auth bypass
- Server-side request forgery via user-controlled URLs

## Disclosure

We aim to ship a fix within 30 days of confirming the issue. After the fix is released, a security advisory will be published. Reporters who want credit will be named in the advisory.

## Supply-chain notes

- All commits on `main` of every submodule are SSH-signed. Anything not signed should be treated with suspicion.
- Submodule pointers in this super-repo are bumped only after the new submodule SHA passes CI.
- `.github/workflows/` runs are pinned to specific action SHAs where possible; the long term plan is to move all `actions/checkout@v4`-style pins to commit SHAs.
