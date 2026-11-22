# AGENTS.md

## What this repo is

`frappe_docker` — Docker images and orchestration for the Frappe/ERPNext stack. It contains **no Frappe application code**; only Dockerfiles, compose configs, CI, tests, and docs. `docs/` is the canonical reference for every workflow — check it before guessing.

## Layout

- `compose.yaml` — production base compose file; `pwd.yml` — disposable demo stack.
- `overrides/` — composable compose overrides (proxy, mariadb/postgres, redis, https, etc.).
- `images/` — Dockerfiles: `production/` (main app images), `bench/`, `custom/`, `layered/`.
- `docs/` — VitePress docs site; `resources/` — nginx/traefik config templates.
- `.github/scripts/` — `get_latest_tags.py`, `update_example_env.py`, `update_pwd.py` keep version pins in `example.env` and `pwd.yml` in sync with releases.

## Commands

- **Lint (CI gate):** `pre-commit run --all-files` — black, isort, prettier, shfmt, shellcheck, codespell. The shfmt hook requires Go to be installed (see `.github/workflows/lint.yml`).
- **Build images:** `FRAPPE_VERSION=... ERPNEXT_VERSION=... docker buildx bake <target>` — targets defined in `docker-bake.hcl` (`bench`, `bench-test`, `base`, `build`, `erpnext`).
- **Integration tests:** from repo root, `pip install -r requirements-test.txt`, then `pytest`. Tests spin up a full Docker Compose stack (`compose.yaml` + proxy/mariadb/redis overrides, plus `tests/compose.ci.yaml` when `CI` is set) — slow and requires Docker. `FRAPPE_VERSION`/`ERPNEXT_VERSION` env vars select versions to test; `develop` maps to image tag `latest` (see `tests/conftest.py`).
- **Docs dev server:** `pnpm docs:dev` / `pnpm docs:build` inside `docs/` (VitePress, pnpm via `packageManager` field).

## Conventions

- Conventional Commits (`fix:`, `feat:`, `docs:`, `chore:`, ...). Branch names: `feature/`, `fix/`, `docs/`.
- Docs pages need VitePress frontmatter with a `title` field (used for the sidebar label); keep markdown GitHub-compatible.
- CI builds and integration-tests images on PRs touching build-relevant paths — image builds are slow and resource-heavy; verify builds locally before pushing repeated commits.

## `development/` environment (where agent sessions usually start)

- `development/` is **git-ignored** except `README.md`, `installer.py`, `apps-example.json`, and `vscode-example/`. Everything else there (benches, sites, node_modules) must never be committed.
- `development/frappe-bench/` is a live bench with real apps (e.g. `frappe`, `insights`, custom apps). Sites live under `development/frappe-bench/sites/`; site names must end in `.localhost`.
- `development/installer.py` bootstraps a bench + site (`-j apps.json -b bench-name -s site.localhost`); `apps-example.json` is the apps template.
- `development/package.json` holds frontend tooling (Vite/Vue/TypeScript) for custom-app work done inside the bench.