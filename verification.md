# Verification Trail — Financial Metrics Dashboard

> Repository: `4GeeksAcademy/ai-eng-financial-dashboard-context-project` (branch: `main`)
> Audit date: 2026-09-04

## Architecture (sourced from code & config)

| Component | Entry point | Port(s) | Source file |
|-----------|-------------|---------|-------------|
| Backend | `python -m debugpy ... -m uvicorn app.main:app --port 8000 --reload` | 8000 (API), 5678 (debug) | `backend/Dockerfile` |
| Frontend | `npm run dev -- --host 0.0.0.0 --port 5173` | 5173 (Vite dev) | `frontend/Dockerfile` |
| API proxy | Vite proxies `/api/*` → `http://backend:8000` | — | `frontend/vite.config.ts` |
| Orchestration | Docker Compose — frontend depends on backend | 5173, 8000, 5678 | `docker-compose.yml` |

## Backend (FastAPI / Python 3.13)

- **9 GET endpoints** in `backend/app/routes.py`: `/health`, `/api/metrics`, `/api/metrics/facets`, `/api/metrics/summary`, `/api/metrics/categories/top`, `/api/metrics/comparison`, `/api/metrics/alerts`, `/api/metrics/b2b`, `/api/metrics/b2c`
- **Data source:** In-memory mock generator (`seed=42`, 360 movements/year). No database.
- **CORS:** `allow_origins=["*"]` (`backend/app/main.py`)
- **Tests:** 8 pytest tests in `backend/tests/test_routes.py`

## Frontend (React 19 / TypeScript 6 / Vite 8)

- **Entry:** `frontend/src/main.tsx` → `App.tsx`
- **UI:** 4 KPI cards (`KPIRow` + `KPICard`) + income/outcome line chart + profit margin line chart (Recharts)
- **Client-side logic:** `computeKPIs()`, `computeMonthlyData()` in `frontend/src/lib/financial-utils.ts`
- **Tests:** 3 Vitest tests in `frontend/src/lib/financial-utils.test.ts`

## Gaps found (confirmed absent — not assumptions)

| Item | Status | How verified |
|------|--------|--------------|
| `.env.example` | **Missing** — referenced in README but does not exist | `file_search("**/.env*")` → 0 results |
| `frontend/src/lib/mock-data.ts` | **Unused** — file exists but imported by zero modules | Code review + grep |
| `.agents/` and `memory-bank/` | **Missing** — referenced in AGENTS.md and README | `list_dir()` → ENOENT |
| CI/CD pipeline | **Not found** | File search for workflows |
| Production build config | **Not found** — Dockerfile runs `npm run dev` (dev server) | `frontend/Dockerfile` |
| Database / persistence | **None** — all data is in-memory mock | `requirements.txt` + `routes.py` |