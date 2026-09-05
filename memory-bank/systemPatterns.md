# System Patterns

> Overall system architecture, technical design patterns, folder structure conventions, and component relationships.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                   Docker Compose (docker-compose.yml)             │
│                                                                   │
│  ┌──────────────────────┐         ┌──────────────────────────┐   │
│  │    Frontend Service   │         │     Backend Service       │   │
│  │   (Vite + React 19)   │ ────── │   (FastAPI + uvicorn)     │   │
│  │   Port 5173 (host)    │  /api/  │   Port 8000 + 5678       │   │
│  │   Port 5173 (cont.)   │  proxy  │   (debugpy)              │   │
│  └──────────┬───────────┘         └──────────────┬────────────┘   │
│             │                                     │               │
│             ▼                                     ▼               │
│    ┌───────────────┐                   ┌──────────────────┐      │
│    │  shadcn/ui     │                   │  Pydantic models  │      │
│    │  components    │                   │  (in routes.py)   │      │
│    │  + Recharts    │                   │                   │      │
│    └───────────────┘                   │  Pure helpers      │      │
│                                         │  (in routes.py)   │      │
│                                         │                   │      │
│                                         │  Mock data gen    │      │
│                                         │  seed=42, 360/mo  │      │
│                                         └──────────────────┘      │
└────────────────────────────────────────────────────────────────────┘
```

## Folder Structure Convention

```
/
├── backend/                        # Python backend
│   ├── Dockerfile                  # Container definition (python:3.13-slim)
│   ├── requirements.txt            # All Python dependencies
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                 # FastAPI app factory + CORS middleware
│   │   └── routes.py               # ALL models, helpers, routes (390 lines)
│   └── tests/
│       ├── conftest.py             # sys.path setup for test imports
│       └── test_routes.py          # Unit + integration tests
├── frontend/                       # TypeScript/React frontend
│   ├── Dockerfile
│   ├── package.json                # Deps: React 19, Recharts, shadcn/ui, Vitest
│   ├── vite.config.ts              # Vite config: proxy, @/ alias, Tailwind
│   ├── tsconfig.app.json           # Strict TS: noUnusedLocals, noUnusedParameters
│   ├── eslint.config.js            # ESLint: .ts/.tsx only
│   ├── components.json             # shadcn/ui configuration
│   ├── public/
│   └── src/
│       ├── main.tsx                # React entry point
│       ├── App.tsx                 # Root component: data fetching + layout
│       ├── index.css               # Tailwind CSS 4 entry
│       ├── assets/
│       ├── components/
│       │   ├── dashboard/
│       │   │   ├── dashboard-header.tsx
│       │   │   ├── kpi-card.tsx
│       │   │   ├── kpi-row.tsx
│       │   │   ├── income-outcome-chart.tsx
│       │   │   └── profit-percent-chart.tsx
│       │   └── ui/                 # shadcn/ui primitives
│       │       ├── card.tsx
│       │       └── skeleton.tsx
│       └── lib/
│           ├── financial-types.ts   # Shared TypeScript types
│           ├── financial-utils.ts   # Pure computation functions
│           ├── financial-utils.test.ts
│           ├── mock-data.ts         # CONFIRMED UNUSED — 52 hardcoded entries
│           └── utils.ts             # cn() utility (tailwind-merge + clsx)
├── docker-compose.yml
├── .agents/
│   └── rules/
│       └── proposed-rule-set.md
├── memory-bank/                    # This documentation
└── verification.md
```

## Design Patterns

### Backend Patterns

1. **Pure Helper Functions** — All data transformation, filtering, aggregation, and calculation is extracted into standalone typed functions (lines 61–245 of `routes.py`). No function has HTTP dependencies, making them unit-testable without `TestClient`.

   ```python
   # Pattern: function(input: typed_params) -> typed_output
   def filter_movements_by_date(
       movements: list[FinancialMovement],
       start_date: date | None,
       end_date: date | None,
   ) -> list[FinancialMovement]:
   ```

2. **Deterministic Mock Data** — `generate_mock_movements(seed=42)` produces the same 360-movement list on every call with the same seed. Used by all 8 route handlers and both test functions.

3. **Query Parameters** — All route handlers use FastAPI's `Query(default=None)` for optional filters and `Query(...)` for required parameters. Type validation is handled by Pydantic `Literal` types (`OperationType`, `Category`, `BusinessType`, `GroupBy`).

4. **No Database / In-Memory** — The entire backend operates on a generated list. No SQL, ORM, or external storage.

### Frontend Patterns

1. **Pure Computation Functions** — `computeKPIs()` and `computeMonthlyData()` in `financial-utils.ts` are pure functions that accept `FinancialMovement[]` and return computed metrics. These are the only data transformations used.

2. **State at the Top** — `App.tsx` is the single stateful component. It fetches data once via `useEffect`, computes KPIs and monthly data, passes them down as props. No state management library.

3. **Loading Skeleton** — Every dashboard component (`KPICard`, `KPIRow`, `IncomeOutcomeChart`, `ProfitPercentChart`) accepts a `loading` prop and renders skeleton placeholders while data is being fetched.

4. **Error Boundary** — `App.tsx` has a top-level error state. When the API call fails, it displays a styled error banner. There is no error boundary React component.

5. **Named `exort function` Components** — Every component uses `export function ComponentName(...)`, never `export default`. Props defined as `interface ComponentNameProps`.

6. **`@/` Path Alias** — All internal imports use `@/` prefix configured in both `tsconfig.app.json` and `vite.config.ts`.

### Data Flow Pattern

```
User loads page
  → App.tsx useEffect fires
  → fetch() calls /api/metrics (via Vite proxy or VITE_API_BASE_URL)
  → Backend generates data (seed=42) → returns JSON
  → App.tsx:
      → computeKPIs(movements) → { totalIncome, totalOutcome, profit, profitPercent }
      → computeMontlyData(movements) → [{ month, income, outcome, profitPercent }]
  → Render:
      DashboardHeader          (period string)
      KPIRow                  (KPIMetrics | null → 4 KPICards)
      IncomeOutcomeChart      (MontlyDataPoint[] → Recharts LineChart)
      ProfitPercentChart      (MontlyDataPoint[] → Recharts LineChart)
```

## Component Dependency Tree

```
App
├── DashboardHeader ← DashboardHeaderProps { period? }
├── KPIRow            ← KPIRowProps { metrics, loading? }
│   ├── KPICard       ← KPICardProps { label, value, helperText, icon, variant, loading? }
│   ├── KPICard       ← same
│   ├── KPICard       ← same
│   └── KPICard       ← same
├── IncomeOutcomeChart ← IncomeOutcomeChartProps { data, loading? }
│   └── CustomTooltip  (private, inline)
└── ProfitPercentChart ← ProfitPercentChartProps { data, loading? }
    └── CustomTooltip  (private, inline)
```

## Route Handler Pattern

Every route handler follows the same structure:
1. Generate mock data: `movements = generate_mock_movements(seed=42)`
2. Optionally filter: `if business_type is not None: movents = [...]`
3. Applydate/category/opration filters:`iltered = filter_movements(...)`
4. Transfom/return:`eturn summarize_movements(filtered, group_by)` (or equivalent)

## CORS Middleware Pattern

`backend/app/main.py` applies CORS as a middleware wrapping the entire router:
```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
app.include_router(router)
```

## Docker Composition Pattern

- Frontend container mounts `./frontend:/app` with `node_modules` shadow volume (`- /app/node_modules`)
- Backend container mounts `./backend:/app`
- Frontend depends on backend (`depends_on: backend`)
- Vite proxies `/api` → `http://backend:8000` inside the compose network
- debugpy port 5678 exposed from backend (for remote debugging)