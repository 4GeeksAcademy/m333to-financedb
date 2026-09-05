# Proposed Rule Set — Financial Metrics Dashboard

> Auto-generated from a full repository audit. Every rule references concrete file paths and line numbers from this repository. Generic engineering advice is intentionally excluded.

---

## Architecture

### ARCH-01: Separate Pydantic Models, Helper Functions, and Route Handlers into Distinct Files

- **Scope:** All Python files under `backend/app/`
- **Rationale:** `backend/app/routes.py` lines 19–59 define 7 Pydantic `BaseModel` subclasses, lines 61–245 define 10 helper functions, and lines 247–390 define 9 route handlers — all in a single ~390-line file. No other Python file under `backend/app/` contains business logic. Importing any route handler pulls in the entire module including all models and helpers.
- **Project-Specific Guidance:** BEFORE adding a new model, helper, or route handler, create separate files: `schemas.py` for Pydantic models, `services.py` or `utils.py` for pure helper functions, and keep `routes.py` for route handlers only. Import them using `from app.schemas import ...`, `from app.services import ...`, etc.
  **Migration path for existing code:** If you are adding new models/helpers while existing ones remain in `routes.py`, place ONLY new code in `schemas.py`/`services.py` — but ALSO migrate the EXISTING models from `routes.py` into `schemas.py` in the same commit, and update all import references. Do NOT leave model classes or helper functions scattered across two files. Either everything is in `routes.py` (no migration done yet), or everything is properly split (full migration done in one change).

### ARCH-02: Extract Duplicated Business-Type Filter Logic into a Single Shared Helper Function

- **Scope:** Only `backend/app/routes.py`
- **Rationale:** The block `if business_type is not None: movements = [item for item in movements if item.business_type == business_type]` is duplicated verbatim at lines 277–279 (`get_metrics_summary`), 295–297 (`get_top_categories`), 311–313 (`get_metrics_comparison`), and 350–352 (`get_metrics_alerts`). No shared `filter_by_business_type` helper exists.
- **Project-Specific Guidance:** Create a single function — e.g., `def filter_by_business_type(movements: list[FinancialMovement], business_type: BusinessType | None) -> list[FinancialMovement]` — and call it from all 4 route handlers. Adding a 5th inline copy of this block is NOT allowed.

### ARCH-03: Define the Mock-Data Seed in Exactly One Place, Not Eight

- **Scope:** `backend/app/services.py` (or whichever file hosts the data-access helpers). When `services.py` does not yet exist, the seed lives temporarily in `backend/app/routes.py` until ARCH-01's migration is complete.
- **Rationale:** `generate_mock_movements(seed=42)` appears 8 separate times in `routes.py`: at lines 255, 264, 277, 295, 311, 350, 370, and 386. Every route handler independently regenerates the same 360-movement list from scratch.
- **Project-Specific Guidance:** Define a module-level factory, e.g., `_get_movements = lambda: generate_mock_movements(seed=42)` in the data-access file. Import and call it from every route handler. If ARCH-01's migration creates a `services.py`, move the seed factory there (e.g., `_MOVEMENTS_FACTORY = lambda: generate_mock_movements(seed=42)`).
  **Memoization decision:** The factory regenerates data on every call. If performance becomes a concern, replace it with a module-level constant: `MOVEMENTS = generate_mock_movements(seed=42)`. Do NOT mix both patterns — either all callers use the factory (fresh data each time) or all use the constant (shared, cached once at module load).

### ARCH-04: New Backend Modules Must Be Imported Using the `app.` Package Prefix

- **Scope:** All Python files under `backend/tests/` and any new files under `backend/app/`
- **Rationale:** `backend/tests/conftest.py` lines 4–6 inserts `ROOT_DIR` (which resolves to `backend/`) into `sys.path`. Existing tests use `from app.routes import generate_mock_movements` (`backend/tests/test_routes.py` line 6), and the application uses `from app.routes import router` (`backend/app/main.py` line 4). A bare `import new_module` would bypass the `sys.path` setup and fail in tests.
- **Project-Specific Guidance:** MUST use `from app.<module_name> import ...` for any code inside `backend/app/`. Do NOT use bare imports like `import services` or `from services import ...`.
  **Model consolidation:** If you introduce a `schemas.py` file, also migrate ALL existing Pydantic models from `routes.py` into `schemas.py` in the same commit and update all import references. Do not leave model classes scattered across two files — the old `routes.py` definitions and the new `schemas.py` definitions must not coexist.

### ARCH-05: Production Deployments Must Not Use the Single CMD That Exposes debugpy and `--reload`

- **Scope:** `backend/Dockerfile` and `docker-compose.yml`
- **Rationale:** `backend/Dockerfile` line 12: `CMD ["python", "-m", "debugpy", "--listen", "0.0.0.0:5678", "-m", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]`. Port 5678 is exposed in `docker-compose.yml` line 14. Only one Dockerfile exists — no production alternative. `debugpy` opens a remote code-execution socket and `--reload` consumes filesystem watchers unnecessarily.
- **Project-Specific Guidance:** Before any deployment, you MUST provide an alternative: either (a) create a `Dockerfile.prod` that omits `debugpy` and `--reload`, or (b) change the CMD to read from an environment variable (e.g., `CMD ["sh", "-c", "${CMD:-python -m uvicorn app.main:app --host 0.0.0.0 --port 8000}"]`). The current single CMD is acceptable only for local development.

### ARCH-06: CORS Origins Must Be Configurable Per Environment, Not Hardcoded to `["*"]`

- **Scope:** `backend/app/main.py`
- **Rationale:** `backend/app/main.py` lines 8–12 hardcode: `allow_origins=["*"]`, `allow_credentials=True`, `allow_methods=["*"]`, `allow_headers=["*"]`. No environment variable or config file overrides these values. The combination `allow_origins=["*"]` with `allow_credentials=True` trusts any origin with credentialed requests.
- **Project-Specific Guidance:** Replace the hardcoded `allow_origins=["*"]` with an environment-variable-driven value, e.g., `allow_origins=os.getenv("CORS_ORIGINS", "*").split(",")`. Use restrictive origins (e.g., `http://localhost:5173`) in development overrides and specific production domains in deployment.

---

## Naming

### NAM-01: All React Components Must Use Named `export function`, Never `export default`

- **Scope:** All `.tsx` files under `frontend/src/components/`
- **Rationale:** All 5 dashboard components use named exports: `DashboardHeader` (`dashboard-header.tsx:8`), `KPIRow` (`kpi-row.tsx:12`), `KPICard` (`kpi-card.tsx:37`), `IncomeOutcomeChart` (`income-outcome-chart.tsx:56`), `ProfitPercentChart` (`profit-percent-chart.tsx:52`). Zero instances of `export default` exist for components in this project.
- **Project-Specific Guidance:** Define components using `export function ComponentName(...)` syntax. Do NOT use `export default function ComponentName(...)` or `export default () => ...`. This ensures consistent IDE auto-imports and refactoring tooling.

### NAM-02: Every React Component Must Define an `interface ComponentNameProps` for Its Props

- **Scope:** All `.tsx` files under `frontend/src/components/`
- **Rationale:** Every component defines a matching `interface` for props: `KPICardProps` (`kpi-card.tsx:10`), `KPIRowProps` (`kpi-row.tsx:9`), `DashboardHeaderProps` (`dashboard-header.tsx:3`), `IncomeOutcomeChartProps` (`income-outcome-chart.tsx:16`), `ProfitPercentChartProps` (`profit-percent-chart.tsx:16`). None use `type` aliases or inline annotations for props.
- **Project-Specific Guidance:** For every component, declare `interface ComponentNameProps { ... }` and use it as the parameter type: `export function ComponentName({ ... }: ComponentNameProps)`. Do NOT use `type ComponentNameProps = ...` or inline the type in the function signature.

### NAM-03: Private Module-Level Python Functions Must Be Prefixed with an Underscore

- **Scope:** All Python files under `backend/app/`
- **Rationale:** `backend/app/routes.py` uses the `_` prefix for internal helpers: `_year_for_month` (line 61) and `_build_movement` (line 66). Neither is imported or tested directly. Public functions like `generate_mock_movements` (line 94) are imported by `backend/tests/test_routes.py` line 6 and correctly lack the prefix.
- **Project-Specific Guidance:** Prefix any function that is NOT a route handler AND NOT directly imported by tests with a single underscore (e.g., `def _helper_name(...)`). Only omit the underscore for functions that are explicitly part of the testable public API.

### NAM-04: Frontend Unit Tests Must Group Scenarios Using `describe("functionName")` / `it("description")`

- **Scope:** All `.test.ts` files under `frontend/src/`
- **Rationale:** `frontend/src/lib/financial-utils.test.ts` organizes tests by function name: `describe("computeKPIs", ...)` (line 10) with `it("calculates totals and profit values", ...)` (line 11) and `it("returns 0 profitPercent when there is no income", ...)` (line 21); followed by `describe("computeMonthlyData", ...)` (line 36).
- **Project-Specific Guidance:** In every `describe` block, use the exact name of the function under test as the first argument. Inside, use `it` blocks with descriptive phrases that complete the sentence "it should...". This keeps test output readable and clarifies which unit is being validated.

### NAM-05: All User-Facing UI Strings Must Be in English

- **Scope:** All `.tsx` files under `frontend/src/`
- **Rationale:** `frontend/src/App.tsx` line 37 uses Spanish: `"No se pudo cargar la informacion financiera. Revisa la API de backend."` — with a typo ("informacion"). All other UI strings are English: `kpi-row.tsx` lines 17–35 (`"Total Income"`, `"Total Outcome"`, `"Profit"`, `"Profit Margin"`), `dashboard-header.tsx` line 13 (`"Financial Overview"`).
- **Project-Specific Guidance:** MUST write every NEW user-facing string in English. Pre-existing non-English strings are identified as tech debt (e.g., `App.tsx:37` has a Spanish error message with a typo "informacion"). Fix these in a dedicated cleanup task, NOT mixed into unrelated feature changes — unless the change directly touches that string (e.g., modifying the error handling in `App.tsx`). This keeps migrations clean and reviewable.

---

## Testing

### TST-01: All Backend HTTP Route Tests Must Use `fastapi.testclient.TestClient`

- **Scope:** All Python test files under `backend/tests/`
- **Rationale:** `backend/tests/test_routes.py` line 5: `from fastapi.testclient import TestClient`, line 8: `client = TestClient(app)`. Every HTTP endpoint test in that file uses this client. `fastapi` and `httpx` are both listed in `backend/requirements.txt`.
- **Project-Specific Guidance:** Import `TestClient` from `fastapi.testclient`, instantiate it with the existing `app` from `app.main`, and use it for all HTTP assertions. Do NOT use `httpx.Client` directly for route testing — `TestClient` provides FastAPI-native dependency override support.

### TST-02: Every Backend Test Function Must Generate Fresh Mock Data Using `seed=42`

- **Scope:** All Python test files under `backend/tests/`
- **Rationale:** `backend/tests/test_routes.py` lines 13 and 20 both call `generate_mock_movements(seed=42)` independently. No shared mutable movements list exists at module level. The deterministic seed `42` guarantees reproducible results.
- **Project-Specific Guidance:** Inside each test function, call `generate_mock_movements(seed=42)` to obtain a fresh list. Do NOT share a module-level `movements` variable between tests — this prevents order-dependent test failures.

### TST-03: Frontend Test Files Must Be Colocated as `.test.ts` Siblings of Their Source Module

- **Scope:** All test files under `frontend/src/`
- **Rationale:** `frontend/src/lib/financial-utils.test.ts` sits next to its source file `frontend/src/lib/financial-utils.ts`. No `__tests__/` directory exists anywhere in `frontend/src/`. The Vitest config (inherited from Vite defaults) does not configure a separate test directory.
- **Project-Specific Guidance:** For a source file at `frontend/src/path/to/module.ts`, create the test at `frontend/src/path/to/module.test.ts`. Do NOT create or use a `__tests__/` directory.

### TST-04: Frontend Tests Must Import `{ describe, expect, it }` from `"vitest"`, Not Jest

- **Scope:** All `.test.ts` files under `frontend/src/`
- **Rationale:** `frontend/src/lib/financial-utils.test.ts` line 1: `import { describe, expect, it } from "vitest"`. `frontend/package.json` lists `"vitest": "^4.0.1"` as a devDependency. No Jest dependency exists in `package.json`.
- **Project-Specific Guidance:** Import all test utilities from `"vitest"`. Using `"@jest/globals"` or `require("jest")` is NOT allowed — there is no Jest config, Jest runner, or Jest dependency in this project.

### TST-05: Backend Test Files Must Follow the `test_<module>.py` Naming Convention

- **Scope:** All Python test files under `backend/tests/`
- **Rationale:** `backend/tests/test_routes.py` currently mixes unit tests (lines 1–30, testing `generate_mock_movements` and `filter_movements_by_date` directly) and integration tests (lines 32+, using TestClient). There is no existing convention for splitting tests across multiple files. As the backend grows, adding everything to `test_routes.py` creates a monolithic file. Tests for models in `schemas.py` have no natural home there.
- **Project-Specific Guidance:** Create test files named `test_<module>.py` matching the source module under test. For example: `test_routes.py` for route handlers, `test_services.py` for helpers in `services.py`, `test_schemas.py` for model validation in `schemas.py`. Appending all new tests to the existing `test_routes.py` is allowed only when the tested code still lives in `routes.py` (pre-migration). Use `conftest.py` for shared fixtures — currently it only handles `sys.path` setup; you may add a `@pytest.fixture` for the `client` if it is reused across multiple test files.

---

## Developer Experience (DX)

### DX-01: All Internal Frontend Imports Must Use the `@/` Path Alias Instead of Relative Paths

- **Scope:** All `.ts` and `.tsx` files under `frontend/src/`
- **Rationale:** `frontend/src/App.tsx` lines 2–9 use `@/components/...` and `@/lib/...`. The alias is configured in two places: `frontend/tsconfig.app.json` line 11 (`"paths": { "@/*": ["./src/*"] }`) and `frontend/vite.config.ts` lines 18–20 (`resolve: { alias: { "@": path.resolve(__dirname, "./src") } }`). The entire codebase uses this convention.
- **Project-Specific Guidance:** MUST use `@/path/to/module` for ALL imports of files under `frontend/src/`. Do NOT use relative paths like `../../components/ui/card` — they break when files are moved and are inconsistent with the established convention.

### DX-02: ESLint Only Covers `**/*.{ts,tsx}` — Do Not Add Plain JavaScript Files to `frontend/src/`

- **Scope:** All files under `frontend/src/`
- **Rationale:** `frontend/eslint.config.js` line 9: `files: ["**/*.{ts,tsx}"]`. Only TypeScript files are included in the ESLint config, which applies `tseslint.configs.recommended`, `reactHooks.configs.flat.recommended`, and `reactRefresh.configs.vite`. Plain `.js` or `.jsx` files would bypass all checks.
- **Project-Specific Guidance:** Create all new source files as `.ts` or `.tsx`. Do NOT add `.js` or `.jsx` files to `frontend/src/` — they will be invisible to linting and type-checking.

### DX-03: Run `tsc -b` from `frontend/` Before Committing to Surface Strict-Mode Errors That the Dev Server Misses

- **Scope:** `frontend/` directory
- **Rationale:** `frontend/tsconfig.app.json` enables `noUnusedLocals` (line 17), `noUnusedParameters` (line 18), `erasableSyntaxOnly` (line 19), and `noFallthroughCasesInSwitch` (line 20). These strict-mode checks only run during `tsc -b`, NOT during Vite's dev server. The build script in `frontend/package.json` line 7 is `"build": "tsc -b && vite build"`.
- **Project-Specific Guidance:** Before committing, run `cd frontend && tsc -b`. If it fails, fix the reported errors (unused locals, unused parameters, fallthrough cases). Do NOT rely solely on `npm run dev` — it does not enforce these checks.

### DX-04: All New Python Dependencies Must Be Added to `backend/requirements.txt`

- **Scope:** `backend/` directory
- **Rationale:** `backend/requirements.txt` lists all dependencies: `fastapi`, `uvicorn[standard]`, `debugpy`, `pytest`, `pytest-cov`, `httpx`. `backend/Dockerfile` lines 4–5: `COPY requirements.txt ./` and `RUN pip install --no-cache-dir -r requirements.txt`. No other dependency mechanism (e.g., `pyproject.toml`, `Pipfile`, `poetry.lock`) exists.
- **Project-Specific Guidance:** Append any new Python package to `backend/requirements.txt`. Do NOT create a separate dependency file (`Pipfile`, `pyproject.toml`, etc.) unless the Dockerfile is also updated to use it. The Docker build pipeline reads ONLY `requirements.txt`.

### DX-05: Running Frontend Outside Docker Requires Setting `VITE_API_BASE_URL` Environment Variable

- **Scope:** `frontend/` when running directly on the host (via `npm run dev`)
- **Rationale:** `frontend/vite.config.ts` line 12 configures the Vite proxy target as `"http://backend:8000"`. The hostname `backend` only resolves inside the Docker Compose network — it fails with a 502 on the host. `frontend/src/App.tsx` line 10 provides the escape hatch: `const API_BASE_URL = import.meta.env.VITE_API_BASE_URL ?? ""`.
- **Project-Specific Guidance:** When running `npm run dev` directly on the host (not via `docker compose`), set `VITE_API_BASE_URL=http://localhost:8000` in a `.env` file in `frontend/` or prefix the command with `VITE_API_BASE_URL=http://localhost:8000 npm run dev`. Without this, all API calls will fail because the host cannot resolve `backend`.

### DX-06: The Canonical Way to Run the Full Stack Is `docker compose up --build` from the Repository Root

- **Scope:** Entire repository
- **Rationale:** `docker-compose.yml` defines both services (`frontend` and `backend`) with inter-container networking, volume mounts (backend syncs `./backend:/app`, frontend syncs `./frontend:/app` with a `node_modules` shadow volume), and `depends_on: backend`. `README.md` line 29 documents `docker compose up --build` as the run command.
- **Project-Specific Guidance:** Use `docker compose up --build` from `/workspaces/m333to-financedb/` as the primary method to run the full application. Running components individually (e.g., `uvicorn` directly or `npm run dev` in isolation) is only acceptable for targeted debugging sessions.

### DX-07: Create and Maintain `frontend/.env.example` as Documented in the README

- **Scope:** `frontend/` directory
- **Rationale:** `README.md` line 31 instructs: "copy `frontend/.env.example` to `.env`". A `file_search` for `.env*` returns zero results — the file does not exist. The only environment variable used by the frontend is `VITE_API_BASE_URL` (consumed at `frontend/src/App.tsx` line 10).
- **Project-Specific Guidance:** Create `frontend/.env.example` with at minimum:
  ```
  # Base URL for the backend API. Only needed when running outside Docker Compose.
  VITE_API_BASE_URL=http://localhost:8000
  ```
  This gives contributors a working starting point and validates the README setup steps.