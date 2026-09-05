# Active Context

> Current status, recent modifications, and ongoing work for the Financial Metrics Dashboard.

## Current Status

The project is in **initial development / handover phase**. A complete repository audit has been performed, resulting in:

- A proposed rule set at `.agents/rules/proposed-rule-set.md` (23 rules across Architecture, Naming, Testing, DX)
- A `memory-bank/` folder initialized with this and companion documents
- A `verification.md` file at the repository root documenting findings
- An active pull request: [PR #6](https://github.com/4GeeksAcademy/ai-eng-financial-dashboard-context-project/pull/6) — *"docs: validación de la fase 1 y comprensión del handover"*

## Recent Modifications

None yet — the project has received analysis and documentation but no production code changes. The current state of the codebase is the original scaffold with mock data.

### Analysis Artifacts Created

| File | Purpose |
|---|---|
| `.agents/rules/proposed-rule-set.md` | 23 repository-specific rules for contributors/agents |
| `memory-bank/activeContext.md` | (this file) Current status and focus |
| `memory-bank/productContext.md` | Project purpose and problem definition |
| `memory-bank/systemPatterns.md` | Architecture, patterns, and folder conventions |
| `memory-bank/techContext.md` | Technology stack and runtime configuration |
| `memory-bank/progress.md` | Completion checklist and debt tracker |
| `verification.md` | Audit trail and findings |

## What Is Currently Being Worked On

1. **Audit completion and handover documentation** — the PR #6 is the primary delivery vehicle
2. **Rule set refinement** — the rules have been validated through simulation (adding a `/api/metrics/year-summary` endpoint) and 4 friction points were identified and fixed

## Immediate Next Steps (Priority Order)

1. **Translate Spanish error string** in `frontend/src/App.tsx:37` to English (flagged as tech debt by NAM-05)
2. **Create `frontend/.env.example`** as documented in README but missing (flagged by DX-07)
3. **Extract `business_type` filter duplication** in `backend/app/routes.py` into a shared helper (ARCH-02)
4. **Centralize mock seed value** from 8 locations to one factory (ARCH-03)
5. **Split `backend/app/routes.py`** into `schemas.py`, `services.py`, `routes.py` (ARCH-01)
6. **Replace CORS `["*"]`** with environment-variable-driven origins (ARCH-06)
7. **Remove `frontend/src/lib/mock-data.ts` or add header comment** — it is confirmed dead code with zero imports

## Open Questions / Unknowns

- Is `frontend/src/lib/mock-data.ts` intended for future use, or should it be deleted? It contains 52 hardcoded `FinancialMovement` entries but is not imported anywhere.
- Is there an intent to move from in-memory mock data to a real database? The entire backend is pure in-memory with `generate_mock_movements(seed=42)`.
- Are there deployment targets or production environments planned? The current `backend/Dockerfile` exposes `debugpy` and `--reload` with no production alternative.