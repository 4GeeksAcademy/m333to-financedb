# Tech Context

> Exact technology stack, runtime environments, dependencies, ports, and configuration setups.

## Technology Stack

### Backend

| Technology | Version | Purpose | Source |
|---|---|---|---|
| Python | 3.13 (slim) | Runtime | `backend/Dockerfile:1` |
| FastAPI | latest (via pip) | Web framework | `backend/requirements.txt` |
| uvicorn | latest (standard extras) | ASGI server | `backend/requirements.txt` |
| Pydantic | (FastAPI dependency) | Data models & validation | `backend/app/routes.py:19-59` |
| debugpy | latest | Remote debugger | `backend/requirements.txt`, `backend/Dockerfile:12` |
| pytest | latest | Test runner | `backend/requirements.txt` |
| pytest-cov | latest | Coverage reporting | `backend/requirements.txt` |
| httpx | latest | HTTP client (TestClient dependency) | `backend/requirements.txt` |

### Frontend

| Technology | Version | Purpose | Source |
|---|---|---|---|
| Node.js | (Docker image default) | Runtime | `frontend/Dockerfile` |
| TypeScript | ~6.x | Language | `frontend/package.json` (tsc -b) |
| React | 19.2.4 | UI framework | `frontend/package.json` |
| Vite | ~8.x | Dev server & bundler | `frontend/vite.config.ts` |
| Vitest | ~4.0.1 | Test runner | `frontend/package.json` |
| Recharts | 3.8.1 | Charting library | `frontend/package.json` |
| Tailwind CSS | 4.2.2 | CSS framework | `frontend/package.json`, `frontend/index.css` |
| @tailwindcss/vite | 4.2.2 | Tailwind Vite plugin | `frontend/package.json` |
| shadcn/ui | (via copied components) | UI component primitives | `frontend/src/components/ui/` |
| lucide-react | 1.8.0 | Icons | `frontend/package.json` |
| class-variance-authority | 0.7.1 | Variant management | `frontend/package.json` |
| clsx | 2.1.1 | Class name utility | `frontend/package.json` |
| tailwind-merge | 3.5.0 | Class merging | `frontend/package.json` |
| ESLint | 9.39.4 | Linting | `frontend/eslint.config.js` |
| @vitejs/plugin-react | 6.0.1 | React Vite plugin | `frontend/package.json` |

## Port Mapping

| Service | Container Port | Host Port | Purpose |
|---|---|---|---|
| frontend | 5173 | 5173 | Vite dev server with HMR |
| backend | 8000 | 8000 | FastAPI/uvicorn HTTP API |
| backend | 5678 | 5678 | debugpy remote debugger |

Defined in `docker-compose.yml` lines 6, 13-14.

## Environment Variables

### Frontend (`import.meta.env.VITE_*`)

| Variable | Required? | Default | Purpose |
|---|---|---|---|
| `VITE_API_BASE_URL` | Only outside Docker | `""` (empty = use Vite proxy) | Backend API base URL |
| `VITE_*` (any) | No | — | Standard Vite env prefix |

Consumed at `frontend/src/App.tsx:10`:
```typescript
const API_BASE_URL = import.meta.env.VITE_API_BASE_URL ?? "";
```

### Backend (no env vars currently used)

No environment variables are currently read by the backend. `backend/app/routes.py` and `backend/app/main.py` use no `os.getenv()` calls. All configuration is hardcoded.

## Configuration Files

| File | Purpose |
|---|---|
| `docker-compose.yml` | Multi-service orchestration, networking, volumes, ports |
| `backend/Dockerfile` | Python image, pip install, debugpy + uvicorn CMD |
| `backend/requirements.txt` | Python dependency manifest |
| `frontend/vite.config.ts` | Dev server host, proxy target, `@/` alias, Tailwind plugin |
| `frontend/tsconfig.app.json` | Strict TS config: `paths`, `noUnusedLocals`, `noUnusedParameters`, `verbatimModuleSyntax` |
| `frontend/tsconfig.json` | Root TS config (references `tsconfig.app.json`) |
| `frontend/eslint.config.js` | ESLint: `.ts/.tsx` only, tseslint, react-hooks, react-refresh |
| `frontend/components.json` | shadcn/ui configuration |
| `frontend/package.json` | Scripts: dev, build, lint, test, test:watch, test:coverage |

## Running the Project

### Primary Method (Docker Compose)

```bash
# From repository root:
docker compose up --build
```

- Frontend: http://localhost:5173
- Backend: http://localhost:8000
- Health check: http://localhost:8000/health → `{"status":"ok"}`
- debugpy: attaches to port 5678

### Individual Components (for debugging)

**Backend only:**
```bash
cd backend
pip install -r requirements.txt
python -m uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

**Frontend only (requires backend running):**
```bash
cd frontend
VITE_API_BASE_URL=http://localhost:8000 npm run dev
```

## Running Tests

### Backend
```bash
cd backend
pip install -r requirements.txt
pytest                           # All tests
pytest --cov=app --cov-report=term  # With coverage
```

### Frontend
```bash
cd frontend
npm test                         # vitest run
npm run test:watch               # vitest (watch mode)
npm run test:coverage             # vitest run --coverage
```

## Key Scripts (from `frontend/package.json`)

| Command | Action |
|---|---|
| `npm run dev` | Start Vite dev server (HMR) |
| `npm run build` | `tsc -b && vite build` |
| `npm run lint` | ESLint check |
| `npm run preview` | Vite preview (serve built output) |
| `npm test` | `vitest run` |
| `npm run test:watch` | `vitest` (watch mode) |
| `npm run test:coverage` | `vitest run --coverage` |

## Dependencies Not Found

The following are NOT present:
- **`.env.example`** — referenced in `README.md:31` but does not exist
- **Database** — no SQLite, PostgreSQL, ORM, or migration tool
- **Auth** — no authentication, JWT, sessions, or API keys
- **State management** — no Redux, Zustand, or Context API
- **CSS preprocessor** — no Sass, Less, or PostCSS plugins beyond Tailwind
- **Dependency lock file** — no `package-lock.json` or `requirements.lock` visible