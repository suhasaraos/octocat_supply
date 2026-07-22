---
name: build-and-run
description: Build, run, and test the OctoCAT Supply Chain monorepo (Express API + React/Vite frontend). USE WHEN the user asks to build the project, compile the API or frontend, start the dev servers, seed the database, run tests, or verify their changes still build and pass.
---

# Build & Run — OctoCAT Supply Chain

This is a TypeScript monorepo with two workspaces:

- `api/` — Express + SQLite REST API (runs on `http://localhost:3000`)
- `frontend/` — React + Vite + Tailwind UI (runs on `http://localhost:5173`)

Commands are driven by the root `Makefile`, which wraps the per-workspace npm scripts and works on Windows, macOS, and Linux. Prefer `make` targets; fall back to the raw npm commands if `make` is unavailable.

**Prerequisite:** Node `^20.19.0 || >=22.12.0`.

> **If `make` is not installed** (common on Windows — you'll see `make: The term 'make' is not recognized`), run the raw npm commands shown under each section instead. Each `make` target maps directly to `cd <workspace> && npm run <script>`, so the fallback below reproduces every target. Optionally install `make` first (e.g. `winget install GnuWin32.Make` or `choco install make`) to use the shorter commands.

The raw-command fallback for the full build-and-run flow is:

```powershell
cd api; npm install; cd ../frontend; npm install; cd ..          # make install
cd api; npm run build; cd ../frontend; npm run build; cd ..       # make build
cd api; npm run dev                                              # make dev-api  (in one terminal)
cd frontend; npm run dev                                         # make dev-frontend (in another)
```

## Install dependencies (first time)

```bash
make install
```

Raw equivalent: `cd api && npm install` then `cd frontend && npm install`.

## Build

```bash
make build            # build API then frontend
make build-api        # API only  (tsc)
make build-frontend   # frontend only (tsc -b && vite build)
```

Raw fallback (no `make`): `cd api; npm run build` and `cd frontend; npm run build`.

Always build before running production or DB commands, because those run the compiled output in `api/dist/`.

## Run

```bash
make dev              # start API + frontend together (hot reload)
make dev-api          # API only
make dev-frontend     # frontend only
```

`make dev` auto-seeds the SQLite database and serves the API on port 3000 and the frontend (Vite) on port 5173.

Raw fallback (no `make`): `make dev` runs both servers via `concurrently`. Without `make`, start each in its own terminal — `cd api; npm run dev` and `cd frontend; npm run dev`.

## Database

```bash
make db-init          # create schema (needs a prior `make build`)
make db-seed          # create schema + load sample data
```

Raw fallback (no `make`): `cd api; npm run db:init` and `cd api; npm run db:seed`.

The dev servers seed automatically, so you usually only need these for a production-style run.

## Test

```bash
make test-api         # API unit tests (Vitest)
make test-e2e         # frontend end-to-end tests (Playwright)
```

Raw fallback (no `make`): `cd api; npm run test` and `cd frontend; npm run test:e2e`.

Note: the frontend has **no unit-test script** — its coverage is Playwright E2E only. Avoid `make test` (its `test-frontend` step calls a script that does not exist); run `make test-api` and `make test-e2e` separately instead.

## Lint

```bash
make lint             # eslint both workspaces
make lint-fix         # eslint with --fix
```

## How to verify a change

1. `make build` — confirm both workspaces compile cleanly. Fix any TypeScript errors and rebuild until clean.
2. `make test-api` (and `make test-e2e` for UI flows) — confirm tests pass. Fix failures and re-run until green.
3. Report success only after the build compiles and the relevant tests pass.
