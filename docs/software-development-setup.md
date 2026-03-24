# Software Development Setup

Use this guide to get DeerFlow ready for software development tasks with the smallest amount of setup and the closest match to CI.

## Pick your environment
- **Docker (recommended):** Consistent, pre-baked images. Use `make docker-init` once, then `make docker-start`/`make docker-stop` for hot-reload services on http://localhost:2026.
- **Local:** Faster feedback if you already have the toolchain. Keep nginx running so all traffic flows through the same entrypoint.

## Prerequisites
- Node.js 22+, pnpm, uv, nginx (checked by `make check`)
- Python 3.12+ (used by `uv`)
- Model/API keys for the providers you plan to use

Run the dependency check first:
```bash
make check
```

## Install and configure
1) Generate config on a fresh clone (skips if `config.yaml` exists):
```bash
make config
```

2) Install backend + frontend dependencies:
```bash
make install
```

3) Configure models and keys:
   - Add model entries in `config.yaml` (see `README.md` examples)
   - Put secrets in `.env` (recommended) or export them in your shell, e.g.:
```bash
OPENAI_API_KEY=...
TAVILY_API_KEY=...
INFOQUEST_API_KEY=...
```

## Recommended command order (development loop)
```bash
make check                # verify prerequisites
make install              # ensure deps are current
cd backend && make lint && make test
cd frontend && pnpm lint && pnpm typecheck
BETTER_AUTH_SECRET=local-dev-secret pnpm build   # run when touching frontend build paths
```

## Run the stack locally
```bash
make dev   # starts LangGraph (2024), Gateway (8001), Frontend (3000), nginx (2026)
```
- Unified entrypoint: http://localhost:2026
- Logs: `logs/langgraph.log`, `logs/gateway.log`, `logs/frontend.log`, `logs/nginx.log`
- Stop everything: `make stop`

If you prefer manual control:
```bash
cd backend && make dev       # LangGraph on 2024
cd backend && make gateway   # Gateway on 8001
cd frontend && pnpm dev      # Frontend on 3000
make nginx                   # nginx reverse proxy on 2026
```

## Docker workflow (stable, portable)
```bash
make docker-init   # first run or after image updates
make docker-start  # hot-reload dev stack on 2026
make docker-stop
```
- For sandboxed provisioning/K8s mode, `make docker-start` also launches the provisioner when configured.
- Share pnpm cache on the host for faster rebuilds (handled by `make docker-init`).

## Validation checklist before pushing
- Backend: `cd backend && make lint && make test`
- Frontend: `cd frontend && pnpm lint && pnpm typecheck`
- Frontend build when env/auth/routing changes: `BETTER_AUTH_SECRET=... pnpm build`
- Optional: `make setup-sandbox` to pre-pull the container sandbox image if you rely on containerized tools.
