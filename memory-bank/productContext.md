# Product Context

> Why this project exists, what problem it solves, and how its features are intended to work.

## Purpose

The **Financial Metrics Dashboard** is a full-stack web application designed to visualize and analyze mock financial data. It serves as a technical showcase / learning project — demonstrating how a FastAPI backend + React/TypeScript frontend can work together to present interactive business metrics.

## Problem

Business stakeholders need quick visibility into financial performance indicators:
- Total income, outcome, and net profit
- Profit margin percentages
- Monthly income vs. outcome trends
- Category-level breakdowns (top income/outcome categories)
- Period-over-period comparisons
- Anomaly alerts when monthly expenses spike above historical averages

Without a dashboard, these insights require manual spreadsheet work or direct database queries.

## Solution

The application provides a single-page dashboard that:
1. **Fetches financial movement data** from a REST API (`/api/metrics`)
2. **Computes KPI metrics** (total income, total outcome, profit, profit margin) using pure transformation functions
3. **Renders interactive charts** using Recharts (line charts for income vs. outcome and profit margin percentage)
4. **Displays KPIs** in styled cards with icons and contextual helper text

## How Features Are Intended to Work

### Backend API (FastAPI)

The backend generates deterministic mock data using a seeded random generator (`seed=42`), producing 360 movements per year (30 per month, 12 months). It exposes these endpoints:

| Endpoint | Purpose | Filters |
|---|---|---|
| `GET /health` | Health check | None |
| `GET /api/metrics` | List all movements | date range, category, operation_type |
| `GET /api/metrics/facets` | Available filter values | None |
| `GET /api/metrics/summary` | Aggregated summary | group_by, date range, category, operation_type, business_type |
| `GET /api/metrics/categories/top` | Top N categories by amount | operation_type, limit, date range, business_type |
| `GET /api/metrics/comparison` | Period-over-period net comparison | start_date, end_date, business_type |
| `GET /api/metrics/alerts` | Outcome anomaly alerts | threshold, group_by, date range, business_type |
| `GET /api/metrics/b2b` | Movements filtered to B2B | date range, category, operation_type |
| `GET /api/metrics/b2c` | Movements filtered to B2C | date range, category, operation_type |

All endpoints return JSON. No authentication, rate limiting, or persistent storage is implemented.

### Frontend Dashboard (React/TypeScript)

The frontend is a single-page React application with:
- **Dashboard Header** — title and period indicator
- **KPI Row** — 4 cards: Total Income, Total Outcome, Profit, Profit Margin (with currency formatting and loading skeletons)
- **Income vs. Outcome Chart** — line chart comparing income and outcome trends over time
- **Profit Margin % Chart** — line chart showing profit margin percentage with a 0% reference line
- **Error state** — displayed when the API call fails

### Data Flow

```
Backend (seed=42) → JSON API → Frontend fetch() → computeKPIs() → KPI cards
                                          ↘ computeMonthlyData() → Recharts
```

Data is ephemeral — generated fresh on every backend request. No caching, database, or state management library is used.

## Target Users

- **Learners / evaluators** who want to understand a FastAPI + React + Vite + shadcn/ui project
- **Technical reviewers** assessing code quality, architecture, and conventions of the project
- **Potential contributors** onboarding via the handover documentation

## Non-Goals (Explicit)

- This is NOT production-ready infrastructure (no auth, no database, debugpy exposed in Docker)
- This is NOT a real data pipeline — all data is mocked with `random.seed(42)`
- This is NOT internationalized — UI strings are in English (with one Spanish error string as debt)