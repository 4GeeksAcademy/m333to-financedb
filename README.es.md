# Panel de Métricas Financieras

<!-- hide -->

Por [@marcogonzalo](https://github.com/marcogonzalo) y [otros contribuidores](https://github.com/4GeeksAcademy/ai-eng-financial-dashboard-context-project/graphs/contributors) en [4Geeks Academy](https://4geeksacademy.com/)

[![build by developers](https://img.shields.io/badge/build_by-Developers-blue)](https://4geeks.com)
[![4Geeks Academy](https://img.shields.io/twitter/follow/4geeksacademy?style=social&logo=x)](https://x.com/4geeksacademy)

_These instructions are [available in English](./README.md)._

**Antes de empezar**: 📗 [Lee las instrucciones](https://4geeks.com/es/lesson/como-comenzar-un-proyecto-de-codificacion) sobre cómo comenzar un proyecto de programación.

<!-- endhide -->

---

_Dashboard de métricas financieras con frontend en React + TypeScript y backend en FastAPI._

## Resumen del Producto

El **Panel de Métricas Financieras** es una aplicación web full-stack que visualiza datos financieros simulados a través de tarjetas KPI y gráficos de líneas Recharts.

### Lo que la aplicación hace

- **Obtiene datos de movimientos financieros** desde un backend FastAPI en `/api/metrics` (fuente: `frontend/src/App.tsx:12-16`)
- **Calcula indicadores clave de rendimiento** — ingreso total, egreso total, ganancia neta y margen de ganancia — usando funciones de transformación TypeScript puras (fuente: `frontend/src/lib/financial-utils.ts:15-26`)
- **Renderiza un tablero** con:
  - Cuatro tarjetas KPI (ingresos, egresos, ganancia, margen) con íconos estilizados y texto de ayuda contextual (fuente: `frontend/src/components/dashboard/kpi-row.tsx`)
  - Un gráfico mensual de **Ingresos vs. Egresos** (fuente: `frontend/src/components/dashboard/income-outcome-chart.tsx`)
  - Un gráfico mensual de **Margen de Ganancia %** con línea de referencia cero (fuente: `frontend/src/components/dashboard/profit-percent-chart.tsx`)
- **Proporciona 9 endpoints** montados en el router de `backend/app/routes.py`: `/health`, `/api/metrics`, `/api/metrics/facets`, `/api/metrics/summary`, `/api/metrics/categories/top`, `/api/metrics/comparison`, `/api/metrics/alerts`, `/api/metrics/b2b`, `/api/metrics/b2c`

### Lo que la aplicación NO hace

- No tiene autenticación ni usuarios — la API es completamente abierta (CORS configurado como `["*"]` en `backend/app/main.py:8`)
- No tiene base de datos ni almacenamiento persistente — todos los datos se generan en memoria mediante `generate_mock_movements(seed=42)`, produciendo 360 movimientos deterministas por llamada (fuente: `backend/app/routes.py:147-158`)
- El backend no usa variables de entorno — actualmente lee cero variables `os.getenv()`

## Stack Tecnológico

### Backend (Python)

| Tecnología | Versión | Propósito | Fuente |
|---|---|---|---|
| **Python** | 3.13-slim | Entorno de ejecución | `backend/Dockerfile:1` |
| **FastAPI** | latest (pip) | Framework REST, documentación automática en /docs | `backend/requirements.txt` |
| **Uvicorn** | latest (pip, standard extras) | Servidor ASGI | `backend/requirements.txt` |
| **Pydantic** | (dependencia FastAPI) | Validación de modelos request/response | `backend/app/routes.py:19-59` |
| **debugpy** | latest (pip) | Depurador remoto de Python (puerto 5678) | `backend/requirements.txt` |
| **pytest** | latest (pip) | Framework de pruebas | `backend/requirements.txt` |
| **pytest-cov** | latest (pip) | Reporte de cobertura de pruebas | `backend/requirements.txt` |
| **httpx** | latest (pip) | Cliente HTTP asíncrono (para TestClient de FastAPI) | `backend/requirements.txt` |

### Frontend (TypeScript/React)

| Tecnología | Versión | Propósito | Fuente |
|---|---|---|---|
| **Node.js** | 24 (Alpine) | Entorno de ejecución | `frontend/Dockerfile:1` |
| **TypeScript** | ~6.0 | Lenguaje con modo estricto | `frontend/package.json` |
| **React** | 19.2 | Framework UI | `frontend/package.json` |
| **Vite** | ~8.0 | Servidor de desarrollo (puerto 5173, HMR) y bundler | `frontend/vite.config.ts` |
| **Vitest** | ~4.1 | Ejecutor de pruebas con cobertura | `frontend/package.json` |
| **Recharts** | 3.8 | Librería de gráficos | `frontend/package.json` |
| **Tailwind CSS** | 4.2 | CSS utilitario vía plugin `@tailwindcss/vite` | `frontend/package.json`, `frontend/vite.config.ts:7` |
| **shadcn/ui** | (componentes copiados) | Primitivas UI (Card, Skeleton) | `frontend/src/components/ui/` |

### Estado Actual ✅

- **Backend**: 9 endpoints funcionales con datos mock deterministas (seed=42, 360 movimientos) y 7 modelos Pydantic
- **Frontend**: Tablero completo con 4 tarjetas KPI, 2 gráficos Recharts, estados de carga/error/vacío, tema oscuro
- **Pruebas**: 15 pruebas backend en `backend/tests/test_routes.py` (cubriendo los 9 endpoints, filtros y combinaciones) y 5 casos de prueba frontend en `frontend/src/lib/financial-utils.test.ts` (computeKPIs, computeMonthlyData, formatCurrency, formatPercent)

### Deuda Técnica Conocida ⚠️

| Prioridad | Problema | Ubicación |
|---|---|---|
| P0 | Mensaje de error en español con typo | `frontend/src/App.tsx:37` |
| P0 | `frontend/.env.example` referenciado en README pero no existe | README / archivo faltante |
| P1 | `generate_mock_movements(seed=42)` llamado 8 veces por carga de página | `backend/app/routes.py` |
| P2 | Todos los modelos, helpers y rutas en un solo archivo `routes.py` (~390 líneas) — sin separación `schemas.py`/`services.py` | `backend/app/routes.py` |
| P3 | CORS configurado como `["*"]` con `allow_credentials=True` | `backend/app/main.py:8` |
| P3 | `debugpy` y `--reload` como único CMD de Docker — sin variante de producción | `backend/Dockerfile:12` |
| P3 | `frontend/src/lib/mock-data.ts` con 52 entradas pero sin imports (código muerto) | `frontend/src/lib/mock-data.ts` |
| P3 | Sin pruebas de componente para los 5 componentes del dashboard ni App.tsx | `frontend/src/components/dashboard/` |

### Próximos Pasos Inmediatos

1. Traducir mensaje de error en `App.tsx` a inglés
2. Crear `frontend/.env.example`
3. Extraer filtro duplicado de `business_type` a función compartida
4. Centralizar `seed=42` en una sola fábrica
5. Dividir `backend/app/routes.py` en `schemas.py`, `services.py`, `routes.py`
6. Reemplazar CORS `["*"]` con configuración por variable de entorno
7. Eliminar o documentar el código muerto `mock-data.ts`

## Cómo ejecutar

```bash
docker compose up --build
```

- **Frontend**: http://localhost:5173
- **Backend**: http://localhost:8000
- **Documentación API (Swagger UI)**: http://localhost:8000/docs

### Ejecutar pruebas

**Backend:**
```bash
cd backend && pip install -r requirements.txt && pytest
```

**Frontend:**
```bash
cd frontend && npm install && npm test
```

## Guía para Agentes / Contribuidores

Este repositorio está diseñado para ser explorado por agentes de IA. Archivos clave de referencia:

| Archivo | Propósito |
|---|---|
| `AGENTS.md` | Instrucciones para encontrar reglas, skills y memory-bank |
| `.agents/rules/proposed-rule-set.md` | 23 reglas del repositorio (arquitectura, naming, pruebas, DX) |
| `memory-bank/` | Documentación completa del proyecto (contexto activo, producto, sistema, tecnología, progreso) |

### Flujo de trabajo recomendado

1. Haz un fork de este repositorio a tu cuenta.
2. Abre tu fork en GitHub Codespaces o clónalo.
3. Ejecuta tu agente de IA para inspeccionar frontend y backend.
4. Revisa las reglas existentes y la documentación del memory-bank.
5. Ajusta y valida las reglas mediante una simulación de tarea real.
6. Aplica cambios y documenta hallazgos.

---

Este y muchos otros proyectos son construidos por estudiantes como parte de los [Coding Bootcamps](https://4geeksacademy.com/) de 4Geeks Academy. Encuentra más acerca de los [cursos](https://4geeksacademy.com/es/comparar-programas) de [Ingeniería de IA](https://4geeksacademy.com/es/coding-bootcamps/ingenieria-ia), [Data Science & Machine Learning](https://4geeksacademy.com/es/coding-bootcamps/curso-datascience-machine-learning), [Ciberseguridad](https://4geeksacademy.com/es/coding-bootcamps/curso-ciberseguridad) y [Full-Stack Software Developer con IA](https://4geeksacademy.com/es/coding-bootcamps/programador-full-stack).
