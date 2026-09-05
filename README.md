# Financial Metrics Dashboard

<!-- hide -->

By [@marcogonzalo](https://github.com/marcogonzalo) and [other contributors](https://github.com/4GeeksAcademy/ai-eng-financial-dashboard-context-project/graphs/contributors) at [4Geeks Academy](https://4geeksacademy.com/)

[![build by developers](https://img.shields.io/badge/build_by-Developers-blue)](https://4geeks.com)
[![4Geeks Academy](https://img.shields.io/twitter/follow/4geeksacademy?style=social&logo=x)](https://x.com/4geeksacademy)

_Estas instrucciones están [disponibles en español](./README.es.md)._

**Before you start**: 📗 [Read the instructions](https://4geeks.com/lesson/how-to-start-a-project) on how to start a coding project.

<!-- endhide -->

---

_Financial metrics dashboard with a React + TypeScript frontend and a FastAPI backend._

## Product Overview

The **Financial Metrics Dashboard** is a full-stack web application that visualizes mock financial data through KPI cards and Recharts line charts. It is a student-built project from the [Career Programs](https://4geeksacademy.com/compare-programs) at [4Geeks Academy](https://4geeksacademy.com), as documented in the existing README footer.

### What the application does

- **Fetches financial movement data** from a FastAPI backend at `/api/metrics` (source: `frontend/src/App.tsx:12-16`)
- **Computes key performance indicators** — total income, total outcome, net profit, and profit margin percentage — using pure TypeScript transformation functions (source: `frontend/src/lib/financial-utils.ts:15-26`)
- **Renders a dashboard** with:
  - Four KPI cards (income, outcome, profit, profit margin) with styled icons and contextual helper text (source: `frontend/src/components/dashboard/kpi-row.tsx`)
  - An **Income vs. Outcome** monthly line chart (source: `frontend/src/components/dashboard/income-outcome-chart.tsx`)
  - A **Profit Margin %** monthly line chart with a zero reference line (source: `frontend/src/components/dashboard/profit-percent-chart.tsx`)
- **Provides 9 router-mounted API endpoints** at routes defined in `backend/app/routes.py`: `/health`, `/api/metrics`, `/api/metrics/facets`, `/api/metrics/summary`, `/api/metrics/categories/top`, `/api/metrics/comparison`, `/api/metrics/alerts`, `/api/metrics/b2b`, `/api/metrics/b2c`

### What the application does NOT do

- No authentication or user accounts — the API is fully open (CORS set to `["*"]` in `backend/app/main.py:8`)
- No real database or persistent storage — all data is generated in-memory via `generate_mock_movements(seed=42)`, producing 360 deterministic movements per call (source: `backend/app/routes.py:147-158`)
- No backend environment variable configuration — the backend currently reads zero `os.getenv()` calls

### How data flows

```
User loads page
  → App.tsx useEffect fires
  → fetch() calls /api/metrics (via Vite proxy or VITE_API_BASE_URL)
  → Backend generates 360 movements from seed=42 → returns JSON
  → App.tsx computes KPIs and monthly data via pure functions
  → Renders DashboardHeader + KPIRow + 2 charts
```

---

## Tech Stack

### Backend (Python)

| Technology | Version | Purpose | Source |
|---|---|---|---|
| **Python** | 3.13-slim | Runtime | `backend/Dockerfile:1` |
| **FastAPI** | latest (pip) | REST web framework, auto-docs at /docs | `backend/requirements.txt` |
| **Uvicorn** | latest (pip, standard extras) | ASGI web server | `backend/requirements.txt` |
| **Pydantic** | (FastAPI dependency) | Request/response model validation | `backend/app/routes.py:19-59` |
| **debugpy** | latest (pip) | Remote Python debugger on port 5678 | `backend/requirements.txt`, `backend/Dockerfile:12` |
| **pytest** | latest (pip) | Test framework | `backend/requirements.txt` |
| **pytest-cov** | latest (pip) | Test coverage reporting | `backend/requirements.txt` |
| **httpx** | latest (pip) | Async HTTP client (required by FastAPI TestClient) | `backend/requirements.txt` |

**Backend structure** — all models, helpers, and routes live in a single file `backend/app/routes.py` (~390 lines). The app entry point is `backend/app/main.py` (13 lines) which creates the FastAPI instance, applies CORS middleware, and includes the router.

### Frontend (TypeScript/React)

| Technology | Version | Purpose | Source |
|---|---|---|---|
| **Node.js** | 24 (Alpine) | Runtime | `frontend/Dockerfile:1` |
| **TypeScript** | ~6.0 | Language with strict mode (`noUnusedLocals`, `noUnusedParameters`) | `frontend/package.json` |
| **React** | 19.2 | UI framework | `frontend/package.json` |
| **React DOM** | 19.2 | DOM rendering | `frontend/package.json` |
| **Vite** | ~8.0 | Dev server (port 5173, HMR) and production bundler | `frontend/vite.config.ts` |
| **Vitest** | ~4.1 | Test runner with coverage | `frontend/package.json` |
| **Recharts** | 3.8 | Charting library (LineChart, ResponsiveContainer) | `frontend/package.json` |
| **Tailwind CSS** | 4.2 | Utility-first CSS via `@tailwindcss/vite` plugin | `frontend/package.json`, `frontend/vite.config.ts:7` |
| **shadcn/ui** | (copied components) | UI primitives (Card, Skeleton) | `frontend/src/components/ui/`, `frontend/components.json` |
| **lucide-react** | 1.8 | Icon library (TrendingUp, TrendingDown, DollarSign, etc.) | `frontend/package.json` |
| **ESLint** | 9.39 | Linting with TypeScript-ESLint, react-hooks, react-refresh | `frontend/eslint.config.js` |

### Infrastructure & Tooling

| Tool | Purpose | Source |
|---|---|---|
| **Docker Compose** | Orchestrates frontend + backend containers | `docker-compose.yml` |
| **Docker** | Containerization for both services | `backend/Dockerfile`, `frontend/Dockerfile` |

### Port Mapping

| Service | Container | Host | Purpose |
|---|---|---|---|
| Frontend | 5173 | 5173 | Vite dev server with HMR |
| Backend (API) | 8000 | 8000 | FastAPI HTTP API |
| Backend (debug) | 5678 | 5678 | debugpy remote debugger |

### Key Dependencies

Backend (`backend/requirements.txt`):
```
fastapi
uvicorn[standard]
debugpy
pytest
pytest-cov
httpx
```

Frontend (`frontend/package.json`):
- **Runtime**: react, react-dom, recharts, lucide-react, class-variance-authority, clsx, tailwind-merge
- **Dev**: typescript ~6.0, vite ~8.0, vitest ~4.1, @tailwindcss/vite, eslint 9, typescript-eslint

---

## Current Status

### What Works ✅

**Backend (9 endpoints — all functional):**

| Endpoint | Method | Description |
|---|---|---|
| `/health` | GET | Returns `{"status": "ok"}` |
| `/api/metrics` | GET | Full list of 360 financial movements, filterable by date/category/type |
| `/api/metrics/facets` | GET | Distinct operation types, business types, categories, date range |
| `/api/metrics/summary` | GET | Grouped summaries by day/week/month, with B2B/B2C filtering |
| `/api/metrics/categories/top` | GET | Top N categories by income or outcome |
| `/api/metrics/comparison` | GET | Period-over-period net value comparison with delta and percentage change |
| `/api/metrics/alerts` | GET | Outcome anomaly alerts when monthly expenses exceed threshold over historical average |
| `/api/metrics/b2b` | GET | B2B-only movements |
| `/api/metrics/b2c` | GET | B2C-only movements |

All endpoints use:
- Deterministic mock data (`seed=42`) — same 360 movements every call
- Pydantic response models (7 `BaseModel` subclasses) enforce consistent JSON shapes
- Pure helper functions with no HTTP dependency, making them unit-testable

**Frontend (dashboard renders with observable states):**

- 4 KPI cards with loading skeleton states and styled badges
- 2 Recharts line charts with loading/empty/error states
- Dark theme using Tailwind CSS 4 with OKLCH color tokens
- All 5 components use named exports and typed `interface XProps` patterns
- Vite proxy forwards `/api` to backend in Docker Compose
- Strict TypeScript with `noUnusedLocals`, `noUnusedParameters`, `verbatimModuleSyntax`

**Testing:**

- Backend: 15 tests in `backend/tests/test_routes.py` — covering mock data generation, date filtering, all 9 endpoints (health, metrics, facets, summary, categories/top, comparison, alerts, b2b, b2c), business-type filtering, category/operation-type filters, and multi-filter combinations. All assertions are verifiable against the test file content.
- Frontend: 5 test cases across 3 describe blocks in `frontend/src/lib/financial-utils.test.ts`: `computeKPIs` (2 cases), `computeMonthlyData` (1 case), `formatCurrency/formatPercent` (2 cases).

### Known Gaps & Tech Debt ⚠️

| Priority | Issue | Location |
|---|---|---|
| **P0** | Error message is in Spanish with a typo ("informacion" missing accent) | `frontend/src/App.tsx:37` |
| **P0** | `frontend/.env.example` referenced in README but does not exist | README instructions / missing file |
| **P1** | `generate_mock_movements(seed=42)` is called 8 times per page load (each route handler regenerates data independently) | `backend/app/routes.py` lines 255, 264, 277, 295, 311, 350, 370, 386 |
| **P1** | `business_type` filter block duplicated 4 times across route handlers | `backend/app/routes.py` lines 277-279, 295-297, 311-313, 350-352 |
| **P2** | All models, helpers, and routes in a single ~390-line `routes.py` file — no `schemas.py` or `services.py` separation | `backend/app/routes.py` |
| **P2** | CORS set to `["*"]` with `allow_credentials=True` (security concern) | `backend/app/main.py:8` |
| **P3** | `debugpy` and `--reload` in the only Docker CMD — no production variant | `backend/Dockerfile:12` |
| **P3** | `frontend/src/lib/mock-data.ts` contains 52 hardcoded entries but is imported nowhere (dead code) | `frontend/src/lib/mock-data.ts` |
| **P3** | No component tests for any of the 5 dashboard components or App.tsx | `frontend/src/components/dashboard/` |

### Immediate Next Priorities

1. Translate the Spanish error message in `App.tsx` to English
2. Create `frontend/.env.example` as promised in the README
3. Extract duplicated `business_type` filtering into a shared helper
4. Centralize the `seed=42` constant into a single factory or module-level variable
5. Split `backend/app/routes.py` into `schemas.py`, `services.py`, and keep only route definitions in `routes.py`
6. Replace hardcoded CORS `["*"]` with environment-variable-driven configuration
7. Remove or document dead `mock-data.ts` file

---

## How to Run

### Prerequisites

- Docker and Docker Compose installed
- GitHub Codespaces (recommended) or local environment

### Quick start

```bash
docker compose up --build
```

- **Frontend**: http://localhost:5173
- **Backend API**: http://localhost:8000
- **API documentation (Swagger UI)**: http://localhost:8000/docs

The frontend uses a Vite proxy for `/api` by default, so no extra environment variables are required in Docker Compose or Codespaces.
If you need to target a different backend origin (running outside Docker), copy `frontend/.env.example` to `frontend/.env` and set `VITE_API_BASE_URL`.

### Running tests

**Backend:**
```bash
cd backend
pip install -r requirements.txt
pytest                               # Run all tests
pytest --cov=app --cov-report=term   # With coverage
```

**Frontend:**
```bash
cd frontend
npm install
npm test                            # vitest run
npm run test:watch                  # vitest (watch mode)
npm run test:coverage               # vitest run --coverage
```

### Running individually (without Docker)

**Backend:**
```bash
cd backend
pip install -r requirements.txt
python -m uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

**Frontend (requires backend at a different origin):**
```bash
cd frontend
npm install
VITE_API_BASE_URL=http://localhost:8000 npm run dev
```

---

## Agent / Contributor Guidance

This repository is designed to be explored and extended by AI coding agents. Key reference files:

| File | Purpose |
|---|---|
| `AGENTS.md` | Instructions for agents on where to find rules, skills, and memory bank |
| `.agents/rules/proposed-rule-set.md` | 23 repository-specific rules covering architecture, naming, testing, and developer experience |
| `memory-bank/activeContext.md` | Current status, recent changes, and open questions |
| `memory-bank/productContext.md` | Product purpose and problem definition |
| `memory-bank/systemPatterns.md` | Architecture diagram, design patterns, and component dependency tree |
| `memory-bank/techContext.md` | Full technology stack, ports, env vars, and configuration |
| `memory-bank/progress.md` | Completion checklist, debt tracker, and test coverage summary |
| `verification.md` | Audit trail of initial codebase analysis |

### Recommended workflow

1. Fork this repository to your account.
2. Open your fork in GitHub Codespaces or clone it.
3. Run your AI agent to inspect both frontend and backend.
4. Review the existing rules and memory bank documentation.
5. Refine and validate the rules through a real task simulation.
6. Apply changes and document findings.

---

This and many other projects are built by students as part of the [Career Programs](https://4geeksacademy.com/compare-programs) at [4Geeks Academy](https://4geeksacademy.com). By [@marcogonzalo](https://github.com/marcogonzalo) and [other contributors](https://github.com/4GeeksAcademy/ai-eng-financial-dashboard-context-project/graphs/contributors). Find out more about [AI Engineering](https://4geeksacademy.com/en/coding-bootcamps/ai-engineering), [Data Science & Machine Learning](https://4geeksacademy.com/en/coding-bootcamps/data-science-ml), [Cybersecurity](https://4geeksacademy.com/en/coding-bootcamps/cybersecurity) and [Full-Stack Software Developer with AI](https://4geeksacademy.com/en/coding-bootcamps/full-stack-developer).
