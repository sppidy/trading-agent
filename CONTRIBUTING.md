# Contributing

Thanks for your interest. This repo is a multi-submodule meta-repo; each subfolder is its own git repo. PRs land on individual submodules, then a `Bump <submodule>` commit lands here to point the super at the new SHA.

## Before you start

- Open a GitHub issue describing what you want to change. Saves both of us a wasted PR if the idea doesn't fit.
- For small changes (typo, doc fix, obvious bug), skip straight to the PR.
- This is a paper-trading project. **Do not submit code that places live orders.** PRs adding live-order paths will be closed without review unless they're behind a feature flag that defaults off and the disclaimer is updated accordingly.

## Workflow

1. Fork the **submodule** repo you want to change (not this super-repo).
2. Create a branch from `main`.
3. Make your change; keep the diff minimal. Match existing style.
4. Run the relevant test job (see `.github/workflows/ci.yml` in the super-repo for what CI checks).
5. Open a PR against the submodule's `main`. Reference the issue.
6. Once merged, the super-repo's submodule pointer will be bumped (maintainer task).

## Commit conventions

- Plain prose subject under ~70 chars. Imperative voice ("Fix X", "Add Y").
- Body explains *why*, not *what*. The diff already shows what.
- **Sign your commits** (GPG or SSH). Unsigned PRs may be asked to be re-signed.
- **Do not** add `Co-Authored-By: Claude/Copilot/AI` trailers. If you used an AI assistant, the work is still yours; treat it the same as any other tool you used to type.

## What gets reviewed

- Correctness > performance > style.
- New code paths need at least one test (pytest for Python, unit test for Kotlin/.NET).
- Public API changes need a doc update in `docs/` or the relevant README.
- No new secrets, no private hostnames, no committed `.env`. CI will reject these.

## Security issues

Don't open a public issue for security problems. See [`SECURITY.md`](SECURITY.md).

## Code of conduct

By participating you agree to abide by [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md).
