# Progress

> Checklist tracking what works, what is verified, and what technical debt or missing pieces currently exist.

## Legend

- ✅ **Verified** — confirmed working via code analysis or test output
- ⚠️ **Partial** — exists but has known issues or incomplete coverage
- ❌ **Missing** — should exist per README, conventions, or architecture
- 🐛 **Bug / Tech Debt** — identified issue in existing code

---

## Backend

### What Works ✅

| Item | Evidence |
|---|---|
| FastAPI application boots with CORS | `backend/app/main.py` — `app = FastAPI(...)`, middleware applied |
| All 9 route handlers return HTTP 200 | `backend/app/routes.py` — health, metrics, facets, summary, categories/top, comparison, alerts, b2b, b2c |
| Pydantic model validation on inputs | Type hints and `Literal` constraints on all Query params (`routes.py` lines 14–17) |
| Pydantic response models | 7 `BaseModel` subclasses (`routes.py` lines 19–59) enforce response shape |
| Deterministic mock data | `generate_mock_movements(seed=42)` — 360 movements, same every run |
| Mock data is chronologically sorted | `movements.sort(key=lambda item: item.create_date)` at `routes.py:155` |
| Date range filtering | `filter_movements_by_date()` — inclusive range tested in `test_routes.py:20-28` |
| Category / operation_type filtering | `filter_movements()` combines all optional filters (`routes.py:171-191`) |
| Group-by aggregation (day/week/month) | `summarize_movements()` with ISO week and year-month keys (`routes.py:212-234`) |
| Health endpoint returns `{"status":"ok"}` | Verified in test `test_routes.py:32-36` |
| Test suite passes | 5 tests in `test_routes.py` using `TestClient` + seed=42 |
| Pure functions are unit-testable | All helpers are `list[FinancialMovement] → typed_output`, no HTTP I/O |

### What Has Issues ⚠️

| Item | Issue | Location |
|---|---|---|
| Mock data regenerated 8 times per request | Every route calls `generate_mock_movements(seed=42)` independently | `routes.py` lines 255, 264, 277, 295, 311, 350, 370, 386 |
| `business_type` filter duplicated 4 times | Same 3-line block appears in `get_metrics_summary`, `get_top_categories`, `get_metrics_comparison`, `get_metrics_alerts` | `routes.py` lines 277-279, 295-297, 311-313, 350-352 |
| All models, helpers, routes in one ~390-line file | No `schemas.py`, no `services.py` | `routes.py` |
| CORS hardcoded to `["*"]` with `credentials=True` | Security risk — trusts any origin | `main.py` lines 8-12 |
| debugpy + --reload in single Docker CMD | Remote code execution socket on every start, no production variant | `Dockerfile` line 12 |
| No `if __name__` guard in `main.py` | Cannot run as script directly (minor) | `main.py` line 1 |

## Frontend

### What Works ✅

| Item | Evidence |
|---|---|
| React 19 app renders at root | `frontend/src/main.tsx` — `createRoot`, renders `<App />` |
| Vite dev server boots | `frontend/vite.config.ts` — host 0.0.0.0:5173 |
| Tailwind CSS 4 loads | `frontend/src/index.css` — `@imort "tailwidcss"` |
| `@/` path alias works | `App.tsx` uses `@/components/...`, configured in both `tsconfig.app.json` and `vite.config.ts` |
| KPI computation works | `computeKPIs()` tested in `financial-utils.test.ts` (3 test cases) |
| Monthly data computation works | `computeMonthlyData()` tested in `financial-utils.test.ts` (1 test case with cross-year data) |
| Currency formatting works | `formatCurrency()` — tested, en-US locale, 0 decimals |
| Percentage formatting works | `formatPercent()` — tested, 1 decimal place |
| Loading skeleton states | All 5 dashboard components accept `loading` prop and render `<Skeleton>` |
| Error state shown on fetch failure | `App.tsx:28-30` catches errors, displays error banner |
| All 5 components use named exports | `DashboardHeader`, `KPIRow`, `KPICard`, `IncomeOutcomeChart`, `ProfitPercentChart` |
| All 5 components define `interface XProps` | Consistent `interface ComponentNameProps` pattern |
| Vite proxy to backend works inside Docker | `vite.config.ts:12` — `target: "http://backend:8000"` |
| ESLint configured for `.ts/.tsx` | `eslint.config.js:9` |
| Strict TypeScript checks enabled | `tsconfig.app.json` — `noUnusedLocals`, `noUnusedParameters`, `noFallthroughCasesInSwitch` |

### What Has Issues ⚠️

| Item | Issue | Location |
|---|---|---|
| Spanish error message with typo | `"No se pudo cargar la informacion financiera. Revisa la API de backend."` — should be English, "informacion" missing accent | `App.tsx:37` |
| `formatCurrency` truncates cents without comment | Uses `minimumFractionDigits: 0, maximumFractionDigits: 0` — backend produces amounts with `.50`, `.25` but frontend displays `$1,200` with no explanation | `financial-utils.ts:53-58` |

### What Is Missing ❌

| Item | Details |
|---|---|
| **`.env.example`** | README instructs to copy it but file doesn't exist |
| **Tests for dashboard components** | `dashboard-header`, `kpi-card`, `kpi-row`, `income-outcome-chart`, `profit-percent-chart` have zero tests |
| **Test for App.tsx** | Root component has no test |
| **UI component tests** | `card.tsx`, `skeleton.tsx` (shadcn/ui) have no tests |

## Dead / Unused Code

### ❌ `frontend/src/lib/mock-data.ts`

| Property | Value |
|---|---|
| **Lines** | ~128 lines |
| **Exports** | `mockMovements`: `FinancialMovement[]` (52 hardcoded entries) |
| **Imported by** | **ZERO files** — confirmed by `grep_search` across `frontend/src/` |
| **Status** | Dead code. Either delete or add header comment explaining purpose. |

## Docker / DevOps

### What Works ✅

| Item | Evidence |
|---|---|
| Docker Compose builds both services | `docker-compose.yml` defines build contexts |
| Frontend volume mounts with `node_modules` shadow | `- /app/node_modules` prevents host node_modules from shadowing container installs |
| Backend volume mount for live reload | `./backend:/app` enables `--reload` to work |
| `depends_on` ordering | `frontend.depends_on: backend` |

### What Has Issues ⚠️

| Item | Issue | Location |
|---|---|---|
| debugpy port exposed in compose | Port 5678 exposed on host — debugger available to anyone who can reach the port | `docker-compose.yml:14` |
| Single Dockerfile for dev and prod | Contains `debugpy` and `--reload` with no alternative | `Dockerfile:12` |
| No healthcheck for either service | `docker-compose.yml` has no `healthcheck` blocks | |

### What Is Missing ❌

| Item | Reason |
|---|---|
| **`Dockerfile.prod`** | No production Dockerfile without debugpy |
| **`.dockerignore`** | Not present — could optimize build context |
| **Healthcheck blocks** | Neither service has a Docker healthcheck |

## Documentation

### What Exists ✅

| File | Content |
|---|---|
| `README.md` | Project overview, setup instructions, Docker compose command |
| `README.es.md` | Spanish translation of README |
| `AGENTS.md` | Agent/contributor guidelines |
| `verification.md` | Audit trail and findings |
| `.agents/rules/proposed-rule-set.md` | 23 rules for contributors/agents |
| `memory-bank/*` | (this) Active context, product, system, tech, progress |

### What Is Missing ❌

| Item | Reason |
|---|---|
| **`frontend/.env.example`** | README instructions unreferenced file |
| **API documentation** | No OpenAPI/Swagger custom docs or endpoint reference beyond code |
| **Architecture Decision Records (ADRs)** | No record of why seed=42, why debugpy in CMD, why mock-only, etc. |

## Test Coverage Summary

### Backend (5 tests)

| Test | Type | Passes |
|---|---|---|
| `test_generate_mock_movements_returns_full_year_sorted_data` | Unit | ✅ |
| `test_filter_movements_by_date_includes_range_edges` | Unit | ✅ |
| `test_health_endpoint_returns_ok` | Integration | ✅ |
| `test_metrics_endpoint_respects_date_filters` | Integration | ✅ |
| `test_metrics_endpoint_respects_multiple_filters` | Integration | ✅ |

**Missing backend tests:**
- `facets` endpoint
- `summary` endpoint (with group_by variants)
- `categories/top` endpoint (with limit edge cases)
- `comparison` endpoint (with delta_pct edge cases)
- `alerts` endpoint (with threshold edge cases)
- `b2b` and `b2c` endpoints

### Frontend (3 `it` blocks)

| Describe | Test | Passes |
|---|---|---|
| `computeKPIs` | `calculates totals and profit values` | ✅ |
| `computeKPIs` | `returns 0 profitPercent when there is no income` | ✅ |
| `computeMonthlyData` | `returns chronological year-month points with aggregated totals` | ✅ |

**Missing frontend tests:**
- `formatCurrency` formatting edge cases
- `formatPercent` edge cases
- All 5 dashboard components (rendering, loading, error states)
- `App.tsx` data fetching logic

## Known Tech Debt Summary (Prioritized)

| Priority | Debt Item | Effort | Impact |
|---|---|---|---|
| P0 | Spanish error string → English | 5 min | User-facing bug |
| P0 | `frontend/.env.example` missing | 2 min | First-run broken promise |
| P1 | Mock seed hardcoded 8 times | 10 min | Maintenance risk |
| P1 | `business_type` filter duplicated 4 times | 10 min | Maintenance risk |
| P2 | Single 390-line `routes.py` | 1-2 hrs | Maintainability |
| P2 | CORS `["*"]` hardcoded | 15 min | Security |
| P3 | debugpy in production CMD | 30 min | Security |
| P3 | Dead `mock-data.ts` | 5 min | Clarity |
| P3 | No tests for 5 dashboard components | 2-3 hrs | Quality gap |
| P3 | No backend tests for 7 of 9 endpoints | 2-3 hrs | Quality gap |