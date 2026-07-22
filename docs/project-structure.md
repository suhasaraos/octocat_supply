# OctoCAT Supply – Project Structure

This document provides a concise map of the repository layout, key technologies, and the role of each major directory and file.

---

## Repository Layout

```
octocat_supply/
├── api/                        # Express.js REST API (TypeScript)
│   ├── database/
│   │   ├── migrations/         # SQL migration scripts (run in order)
│   │   └── seed/               # SQL seed-data scripts
│   ├── src/
│   │   ├── db/                 # SQLite connection, config, and helpers
│   │   ├── models/             # TypeScript entity model types
│   │   ├── repositories/       # Repository layer (CRUD over SQLite)
│   │   ├── routes/             # Express route handlers (one file per entity)
│   │   ├── utils/              # Shared utilities and error types
│   │   ├── index.ts            # Application entry point
│   │   ├── init-db.ts          # Database initialisation CLI (migrate + seed)
│   │   ├── seedData.ts         # Programmatic seed helpers
│   │   ├── generate-swagger.ts # OpenAPI/Swagger generation script
│   │   └── swagger-options.ts  # Swagger configuration
│   ├── api-swagger.json        # Generated OpenAPI specification
│   ├── ERD.png                 # Entity-relationship diagram
│   ├── Dockerfile              # API container image
│   ├── package.json
│   ├── tsconfig.json
│   └── vitest.config.ts        # Vitest unit-test config
│
├── frontend/                   # React + Vite + Tailwind CSS UI (TypeScript)
│   ├── public/                 # Static assets (hero image, icons)
│   ├── src/
│   │   ├── api/                # Axios client config (base URL, interceptors)
│   │   ├── assets/             # Images and icons imported by components
│   │   ├── components/
│   │   │   ├── entity/         # Per-entity CRUD views (e.g. product/)
│   │   │   ├── admin/          # Admin-specific UI components
│   │   │   ├── About.tsx
│   │   │   ├── Footer.tsx
│   │   │   ├── Login.tsx
│   │   │   ├── Navigation.tsx
│   │   │   └── Welcome.tsx
│   │   ├── context/            # React context providers
│   │   ├── styles/             # Global CSS / Tailwind base styles
│   │   ├── App.tsx             # Root component and router setup
│   │   └── main.tsx            # React DOM entry point
│   ├── tests/                  # Playwright end-to-end tests
│   ├── index.html
│   ├── Dockerfile              # Frontend container image (nginx)
│   ├── nginx.conf              # nginx config for production serving
│   ├── vite.config.ts
│   ├── tailwind.config.js
│   ├── playwright.config.ts
│   └── package.json
│
├── docs/                       # Project documentation
│   ├── architecture.md         # System architecture overview + Mermaid diagrams
│   ├── build.md                # Build, run, and test instructions
│   ├── sqlite-integration.md   # SQLite integration guide (migrations, repos, errors)
│   ├── tao.md                  # Design philosophy / TAO document
│   ├── project-structure.md    # ← this file
│   └── design/                 # UI design references and mockups
│
├── .github/
│   ├── copilot-instructions.md # Repo-wide Copilot guidance
│   └── instructions/           # Path-scoped Copilot instruction files
│       ├── api.instructions.md
│       ├── database.instructions.md
│       └── frontend.instructions.md
│
├── .devcontainer/              # Dev Container configuration
├── .vscode/                    # VS Code tasks and launch configurations
├── docker-compose.yml          # Compose file: api (port 3000) + frontend (port 3001)
├── Makefile                    # Top-level developer workflow targets
├── .nvmrc                      # Pinned Node.js version (22.x recommended)
└── README.md                   # Quick-start guide
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | React 19, TypeScript, Vite 8, Tailwind CSS 4, React Query (TanStack), React Router 7 |
| **Backend** | Express 5, TypeScript, better-sqlite3 |
| **Database** | SQLite (file: `api/data/app.db`; in-memory for tests) |
| **API Docs** | Swagger / OpenAPI (swagger-jsdoc + swagger-ui-express) |
| **Testing** | Vitest (unit), Playwright (e2e) |
| **Linting** | ESLint 10 + typescript-eslint |
| **Containerisation** | Docker + Docker Compose |
| **Node.js** | 20.19+ / 22.13+ (see `.nvmrc`) |

---

## Data Model

The eight core entities and their relationships (see also `api/ERD.png`):

```
Headquarters ──< Branch ──< Order ──< OrderDetail >── Product
                                          │
                                  OrderDetailDelivery
                                          │
                           Supplier ──< Delivery
```

Corresponding database tables: `headquarters`, `branches`, `orders`, `order_details`, `products`, `suppliers`, `deliveries`, `order_detail_deliveries`, `migrations`.

---

## API Layer (`api/src/`)

| Directory / File | Purpose |
|---|---|
| `db/` | SQLite connection factory, config (`DB_FILE`, WAL mode, foreign keys), error types |
| `models/` | One TypeScript interface per entity (`Product`, `Order`, etc.) |
| `repositories/` | Repository classes with typed `findAll`, `findById`, `create`, `update`, `delete` methods; snake_case ↔ camelCase mapping |
| `routes/` | Express routers — one file per entity, mounted in `index.ts`; JSDoc annotations feed Swagger |
| `utils/` | Shared helpers and custom error classes (`NotFoundError`, `ValidationError`, `ConflictError`) |
| `index.ts` | Bootstraps Express, mounts routes, registers Swagger UI, starts HTTP server |
| `init-db.ts` | CLI script: runs SQL migrations in `database/migrations/` then optionally seeds from `database/seed/` |

---

## Frontend Layer (`frontend/src/`)

| Directory / File | Purpose |
|---|---|
| `api/config.ts` | Axios instance configured from `VITE_API_URL` environment variable |
| `components/entity/` | Entity-specific pages (list, detail, create/edit forms) — one sub-folder per entity |
| `components/admin/` | Admin dashboard components |
| `components/*.tsx` | Shared layout components (Navigation, Footer, Welcome, Login, About) |
| `context/` | React context providers (e.g. auth state) |
| `App.tsx` | React Router routes wired to page components |
| `main.tsx` | ReactDOM render root; wraps app in QueryClientProvider |

---

## Database Migrations & Seed Data

Migration SQL files in `database/migrations/` are numbered sequentially and executed once (tracked in the `migrations` table). Seed SQL files in `database/seed/` populate demo data.

```
database/migrations/001_init.sql                     # Full schema
database/migrations/002_add_supplier_status_fields.sql
database/seed/001_suppliers.sql
database/seed/002_headquarters.sql
database/seed/003_branches.sql
database/seed/004_products.sql
```

---

## Developer Workflow

```bash
# Install all dependencies
make install

# Start API (port 3000) + frontend (port 5137) in watch mode
make dev

# Run all tests
make test

# Build for production
make build

# Database management
make db-init      # migrations + seed
make db-migrate   # migrations only
make db-seed      # seed only

# Lint both workspaces
make lint

# Docker Compose (API on 3000, frontend on 3001)
docker-compose up --build
```

Environment variables for the API:

| Variable | Default | Description |
|---|---|---|
| `DB_FILE` | `./data/app.db` | SQLite database file path |
| `DB_ENABLE_WAL` | `true` | Enable WAL journal mode |
| `DB_FOREIGN_KEYS` | `true` | Enforce foreign key constraints |
| `DB_TIMEOUT` | `30000` | Busy timeout (ms) |

Frontend environment variable:

| Variable | Description |
|---|---|
| `VITE_API_URL` | Base URL for the backend API (e.g. `http://localhost:3000`) |

---

## Related Documentation

- [`docs/architecture.md`](architecture.md) – Architecture overview with Mermaid component and ERD diagrams
- [`docs/sqlite-integration.md`](sqlite-integration.md) – Deep-dive into the repository pattern, migrations, and error handling
- [`docs/build.md`](build.md) – Detailed build, run, and test reference
- [`api/api-swagger.json`](../api/api-swagger.json) – Generated OpenAPI 3.0 specification
