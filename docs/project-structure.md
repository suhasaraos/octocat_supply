# OctoCAT Supply Chain — Project Structure

This document provides an overview of the repository layout and the purpose of each major file and directory.

## Repository Root

```
octocat_supply/
├── .devcontainer/          # Dev Container configuration for VS Code
├── .github/                # GitHub configuration (Copilot instructions, agents, workflows)
├── .vscode/                # VS Code workspace settings and launch configurations
├── api/                    # Express.js backend API
├── docs/                   # Project documentation
├── frontend/               # React + Vite frontend application
├── .gitignore
├── .nvmrc                  # Node.js version pin (22.13 recommended)
├── docker-compose.yml      # Multi-container Docker Compose definition
├── Makefile                # Cross-platform task runner (install, dev, build, test, docker…)
└── README.md               # Getting-started guide
```

---

## `api/` — Backend (Express.js + SQLite)

```
api/
├── database/
│   ├── migrations/         # Ordered SQL migration scripts
│   │   ├── 001_init.sql
│   │   └── 002_add_supplier_status_fields.sql
│   └── seed/               # Ordered SQL seed-data scripts
│       ├── 001_suppliers.sql
│       ├── 002_headquarters.sql
│       ├── 003_branches.sql
│       └── 004_products.sql
├── src/
│   ├── db/                 # Database infrastructure
│   │   ├── config.ts       # DB environment-variable configuration
│   │   ├── migrate.ts      # Migration runner
│   │   ├── seed.ts         # Seed runner
│   │   └── sqlite.ts       # SQLite connection factory (file or :memory:)
│   ├── models/             # TypeScript entity types (camelCase)
│   │   ├── branch.ts
│   │   ├── delivery.ts
│   │   ├── headquarters.ts
│   │   ├── order.ts
│   │   ├── orderDetail.ts
│   │   ├── orderDetailDelivery.ts
│   │   ├── product.ts
│   │   └── supplier.ts
│   ├── repositories/       # Repository pattern — thin DB access layer
│   │   ├── branchesRepo.ts
│   │   ├── deliveriesRepo.ts
│   │   ├── headquartersRepo.ts
│   │   ├── orderDetailDeliveriesRepo.ts
│   │   ├── orderDetailsRepo.ts
│   │   ├── ordersRepo.ts
│   │   ├── productsRepo.ts
│   │   ├── suppliersRepo.ts
│   │   └── suppliersRepo.test.ts
│   ├── routes/             # Express route handlers (one file per entity)
│   │   ├── branch.ts
│   │   ├── branch.test.ts
│   │   ├── delivery.ts
│   │   ├── headquarters.ts
│   │   ├── order.ts
│   │   ├── orderDetail.ts
│   │   ├── orderDetailDelivery.ts
│   │   ├── product.ts
│   │   └── supplier.ts
│   ├── utils/
│   │   ├── errors.ts       # Custom error types (NotFoundError, ValidationError, …)
│   │   └── sql.ts          # SQL helper utilities
│   ├── generate-swagger.ts # Script to regenerate api-swagger.json
│   ├── index.ts            # Application entry point (Express setup, middleware)
│   ├── init-db.ts          # DB initialisation on startup
│   ├── seedData.ts         # Programmatic seed-data helpers
│   └── swagger-options.ts  # Swagger/OpenAPI configuration
├── api-swagger.json        # Generated OpenAPI specification
├── ERD.png                 # Entity-Relationship Diagram image
├── Dockerfile              # Container image for the API
├── eslint.config.mjs       # ESLint configuration
├── package.json
├── tsconfig.json
└── vitest.config.ts        # Vitest unit-test configuration
```

### Key API concepts

| Concept | Location | Notes |
|---|---|---|
| REST routes | `src/routes/*.ts` | One file per domain entity |
| Data models | `src/models/*.ts` | TypeScript interfaces, camelCase |
| DB access | `src/repositories/*.ts` | Repository pattern; maps to snake_case columns |
| Migrations | `database/migrations/` | Executed in order; tracked in `migrations` table |
| Seed data | `database/seed/` | SQL scripts for demo data |
| Error types | `src/utils/errors.ts` | `NotFoundError` (404), `ValidationError` (400), `ConflictError` (409) |
| OpenAPI docs | `api-swagger.json` | Regenerated with `make swagger` |

---

## `frontend/` — Frontend (React 18 + Vite + Tailwind CSS)

```
frontend/
├── public/                 # Static assets served as-is (hero image, product images)
├── src/
│   ├── api/
│   │   └── config.ts       # Axios base-URL and API client setup
│   ├── assets/             # Bundled static assets (icons, images)
│   ├── components/
│   │   ├── About.tsx       # About page component
│   │   ├── Footer.tsx      # Site footer
│   │   ├── Login.tsx       # Login form
│   │   ├── Navigation.tsx  # Top navigation bar
│   │   ├── Welcome.tsx     # Landing/welcome page
│   │   ├── admin/
│   │   │   └── AdminProducts.tsx   # Admin product management view
│   │   └── entity/
│   │       └── product/
│   │           ├── Products.tsx    # Product listing
│   │           └── ProductForm.tsx # Create / edit product form
│   ├── context/
│   │   ├── AuthContext.tsx         # Authentication React context
│   │   ├── ThemeContext.tsx        # Dark / light theme context
│   │   ├── themeContextUtils.tsx   # Theme helper utilities
│   │   └── useTheme.tsx            # Custom hook for theme
│   ├── styles/             # Global CSS / Tailwind base styles
│   ├── App.tsx             # Root component (routing, layout)
│   ├── index.css           # Tailwind directives and global resets
│   ├── main.tsx            # Vite entry point (ReactDOM.render)
│   └── vite-env.d.ts       # Vite environment type declarations
├── tests/                  # Playwright end-to-end tests
├── Dockerfile              # Container image for the frontend (nginx)
├── index.html              # HTML shell for Vite
├── nginx.conf              # nginx config for production container
├── playwright.config.ts    # Playwright E2E test configuration
├── tailwind.config.js      # Tailwind CSS configuration
├── vite.config.ts          # Vite build configuration
├── package.json
├── tsconfig.app.json
└── tsconfig.json
```

### Key frontend concepts

| Concept | Location | Notes |
|---|---|---|
| App entry | `src/main.tsx` | Mounts React app |
| Routing | `src/App.tsx` | React Router v6 routes |
| Auth state | `src/context/AuthContext.tsx` | Context + hooks pattern |
| Theme state | `src/context/ThemeContext.tsx` | Dark/light mode |
| API config | `src/api/config.ts` | Base URL from `VITE_API_URL` env var |
| Styling | Tailwind CSS utility classes | Configured in `tailwind.config.js` |
| E2E tests | `tests/` | Playwright; run with `make test-e2e` |

---

## `docs/` — Documentation

```
docs/
├── architecture.md         # High-level architecture overview and ERD
├── build.md                # Build and deployment notes
├── project-structure.md    # This file — repository layout reference
├── sqlite-integration.md   # SQLite setup, repository pattern, testing strategy
├── tao.md                  # Design philosophy / "The Art Of" notes
└── design/                 # UI design assets and mockups
```

---

## `.github/` — GitHub & Copilot Configuration

```
.github/
├── copilot-instructions.md         # Repository-wide Copilot coding guidelines
├── instructions/
│   ├── api.instructions.md         # API-specific Copilot review rules
│   ├── database.instructions.md    # Database/migration Copilot review rules
│   └── frontend.instructions.md    # Frontend Copilot review rules
├── agents/                         # Custom Copilot agent definitions
└── skills/                         # Custom Copilot skill definitions
```

---

## Data Model (ERD)

The domain is a supply chain: suppliers deliver products to branches ordered by headquarters.

```
Headquarters ──< Branch ──< Order ──< OrderDetail >── Product
                                           │
                                    OrderDetailDelivery
                                           │
                              Delivery >── Supplier
```

| Table | Description |
|---|---|
| `suppliers` | External product suppliers |
| `headquarters` | Company HQ records |
| `branches` | Branch offices linked to an HQ |
| `products` | Product catalogue, linked to a supplier |
| `orders` | Orders placed by a branch |
| `order_details` | Line items within an order (order + product) |
| `deliveries` | Shipments from a supplier |
| `order_detail_deliveries` | Junction table linking order line items to deliveries |

---

## Development Workflow

| Task | Command |
|---|---|
| Install dependencies | `make install` |
| Start dev servers (API + frontend) | `make dev` |
| Start API only | `make dev-api` |
| Start frontend only | `make dev-frontend` |
| Build for production | `make build` |
| Run all tests | `make test` |
| Run E2E tests | `make test-e2e` |
| Lint code | `make lint` |
| Initialise database | `make db-init` |
| Seed database | `make db-seed` |
| Regenerate OpenAPI spec | `make swagger` |
| Build Docker images | `make docker-build` |
| Start Docker containers | `make docker-up` |

- API runs on **port 3000** in development.
- Frontend runs on **port 5137** in development.
- Database file: `api/data/app.db` (override with `DB_FILE` env var).
- Node.js version: **22.13** (see `.nvmrc`).
