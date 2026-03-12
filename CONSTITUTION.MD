# Project Constitution — Backend

## Metadata

| Field | Value |
|-------|-------|
| Project Name | AutoMarket — Car Selling Platform |
| Project Type | GREENFIELD |
| Status | DRAFT |
| Author | AI Agency OS / BA Agent |
| Date | 2026-03-12 |
| Stack | Backend |
| Label | Backend |

---

## Overview

AutoMarket is an online car selling platform connecting private sellers and dealerships with buyers. The backend provides REST APIs for vehicle listing management, user accounts, search and filtering, messaging between parties, payment/escrow coordination, and dealership inventory management. It serves the web frontend and mobile app as a single source of truth.

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Language | Python 3.12+ |
| Framework | FastAPI |
| Database | PostgreSQL 16+ |
| ORM | SQLAlchemy 2.0+ (async with asyncpg) |
| Migrations | Alembic |
| Search | Elasticsearch 8.x (hybrid dense + BM25 via RRF) |
| Cache | Redis |
| Auth | Supabase Auth (JWT + RLS) |
| Object Storage | Supabase Storage |
| Task Queue | Celery + Redis broker |
| Testing | pytest + pytest-asyncio + httpx |
| Linting | Ruff (linting + formatting) |
| Type Checking | mypy (strict mode) |
| API Docs | Auto-generated OpenAPI 3.1 via FastAPI |
| Containerization | Docker + Docker Compose |

---

## Environments

| Environment | Purpose | Config Source |
|-------------|---------|--------------|
| development | Local dev, Docker Compose, seeded test data | `.env.development` |
| staging | Pre-production mirror, real integrations, test payments | `.env.staging` via CI secrets |
| production | Live system | `.env.production` via CI secrets / vault |

All configuration via Pydantic Settings reading environment variables. No config files committed to git. Env var prefix: `AUTOMARKET_`.

---

## Coding Standards

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Files / modules | snake_case | `vehicle_listing.py` |
| Functions | snake_case | `get_listing_by_id()` |
| Classes | PascalCase | `VehicleListingService` |
| Variables | snake_case | `listing_count` |
| Constants | UPPER_SNAKE_CASE | `MAX_IMAGES_PER_LISTING` |
| Pydantic models | PascalCase + suffix | `VehicleCreateRequest`, `VehicleResponse` |
| SQLAlchemy models | PascalCase, singular | `Vehicle`, `User`, `Offer` |
| Endpoints | kebab-case in URLs | `/api/v1/vehicle-listings` |
| Env variables | UPPER_SNAKE_CASE, prefixed | `AUTOMARKET_DB_URL` |

### File Structure

```
src/
├── main.py                # App factory, lifespan, middleware
├── config.py              # Pydantic Settings
├── dependencies.py        # Shared FastAPI Depends
├── exceptions.py          # Custom exception classes + handlers
├── models/                # SQLAlchemy ORM models
├── schemas/               # Pydantic request/response schemas
├── api/
│   └── v1/               # Versioned route handlers (thin wrappers)
├── services/              # Business logic (no HTTP, no DB imports)
├── repositories/          # Data access (SQLAlchemy queries)
├── tasks/                 # Celery async tasks
├── middleware/
├── utils/
├── migrations/            # Alembic
tests/
├── conftest.py
├── unit/
├── integration/
├── factories/             # Factory Boy model factories
```

### Patterns to Use

- **Repository pattern** — all database access through `repositories/`. Services never import SQLAlchemy directly.
- **Service layer** — all business logic in `services/`. Endpoints are thin wrappers: validate input, call service, return response.
- **Dependency injection** — FastAPI `Depends()` for DB sessions, current user, permissions. No global state.
- **Async everywhere** — all endpoints, DB queries (asyncpg), and HTTP calls (httpx) are async. No sync blocking in the event loop.
- **Pydantic v2 for all I/O** — strict validation inbound, explicit serialization outbound. `model_config = ConfigDict(from_attributes=True)` for ORM conversion.
- **Explicit error handling** — custom exception classes caught by FastAPI exception handlers. Never return raw 500s.
- **Feature flags** — environment variables or simple DB table for toggles. No hardcoded conditionals for rollout logic.

### Patterns to Avoid

- **Fat endpoints** — if an endpoint exceeds ~20 lines, logic belongs in a service.
- **N+1 queries** — use `selectinload()` / `joinedload()`. Review all list endpoints.
- **Raw SQL in services** — all queries through repositories. Raw SQL only in migrations.
- **Mutable default arguments** — never `def func(items=[])`.
- **Catching bare `Exception`** — catch specific exceptions. Let unexpected errors propagate to the global handler.
- **Business logic in models** — SQLAlchemy models are data containers only.
- **Hardcoded secrets** — all secrets via environment variables. Never in code, never committed to git.

---

## Git Conventions

| Convention | Value |
|------------|-------|
| Branch Naming | `feature/AM-{ticket}-short-desc`, `fix/AM-{ticket}-short-desc`, `chore/`, `hotfix/` |
| Commit Format | Conventional Commits: `feat(vehicles): add search endpoint`, `fix(auth): handle expired refresh token` |
| PR Size Guideline | ≤400 lines changed. Larger PRs must be split or justified. |
| Default Branch | `main` |
| Merge Strategy | Squash merge to `main` |

---

## Error Handling

- All service-layer errors raise custom exceptions inheriting from `AppException(status_code, error_code, message, details)`.
- FastAPI exception handler catches `AppException` and returns: `{"error": {"code": "VEHICLE_NOT_FOUND", "message": "...", "details": {}}}`.
- Unhandled exceptions return generic 500 with no internal details exposed. Full traceback logged server-side.
- Validation errors (Pydantic) return 422 with field-level detail automatically via FastAPI.
- External service failures (Elasticsearch down, Supabase timeout) caught in services, logged with context, and returned as 503 with retry hint.
- No silent exception swallowing. Every `except` block either re-raises, logs, or returns a meaningful error.

---

## Logging & Observability

| Convention | Value |
|------------|-------|
| Log Format | Structured JSON via `structlog` |
| Log Levels | `DEBUG` (local only), `INFO` (request lifecycle, business events), `WARNING` (degraded state, retries), `ERROR` (failures requiring attention) |
| What Gets Logged | Request ID, user ID, action, duration, status code, external service calls, agent-triggered operations |
| What Never Gets Logged | Passwords, JWT tokens, full credit card numbers, PII beyond user ID |
| Tracing | Langfuse for LLM agent traces, OpenTelemetry-compatible request tracing |

Every log line includes `request_id` for correlation. Background tasks (Celery) include `task_id` and `trigger_source`.

---

## Testing Standards

| Requirement | Value |
|-------------|-------|
| Minimum Coverage | 80% (services and repositories) |
| Unit Tests Required | YES |
| Integration Tests Required | YES |

### Test File Conventions

- Test files mirror source structure: `src/services/vehicle_service.py` → `tests/unit/services/test_vehicle_service.py`.
- Prefix all test files with `test_`.
- `pytest-asyncio` for all async tests.
- `httpx.AsyncClient` for integration tests against the FastAPI app.
- Factory Boy for model factories. Never construct test data inline.
- Integration tests use a real PostgreSQL test database (Docker), not SQLite.
- One behavior per test function. Name format: `test_<action>_<condition>_<expected_result>`.
- No test interdependencies. Every test sets up and tears down its own data.

---

## API Standards

| Convention | Value |
|------------|-------|
| Style | REST |
| Versioning | URL prefix: `/api/v1/` |
| Error Format | `{"error": {"code": "VEHICLE_NOT_FOUND", "message": "...", "details": {}}}` |
| Pagination | Cursor-based for feeds/lists, offset for admin views |
| Date Format | ISO 8601 UTC (`2026-03-12T14:30:00Z`) |
| ID Format | UUID v4 |
| Auth Header | `Authorization: Bearer <jwt>` |
| Rate Limiting | Per-endpoint, returned in `X-RateLimit-*` headers |
| Response Envelope | `data` key for success, `error` key for failures |

Specific endpoints are defined by the SA Agent in `design.md`, not in this constitution.

---

## Security Standards

- All endpoints require authentication except: vehicle search, public listing view, registration, login.
- JWT tokens validated via Supabase Auth. Short-lived access tokens (15 min), long-lived refresh tokens.
- Row-Level Security (RLS) at the PostgreSQL layer via Supabase.
- All user input validated by Pydantic before reaching services.
- Image uploads: file type whitelist (JPEG, PNG, WebP), max 10MB, EXIF stripped before storage.
- Rate limiting on auth endpoints (5 attempts/min), search (30 req/min), listing creation (10/hour).
- CORS restricted to known frontend origins. No wildcard `*` in production.
- All inter-service communication over HTTPS. Internal services on private network.
- Secrets via environment variables only.
- SQL injection prevented by parameterized queries (SQLAlchemy ORM).
- Dependency scanning via `pip-audit` in CI.

---

## Documentation Standards

| What | Requirement |
|------|-------------|
| Public functions / methods | Docstring with description, args, returns, raises. Google style. |
| API endpoints | FastAPI auto-docs sufficient. Add `summary` and `description` params to route decorators. Response models fully typed. |
| Complex business logic | Inline comments explaining *why*, not *what*. |
| Architecture decisions | ADR (Architecture Decision Record) in `docs/adr/` for non-trivial choices. |

---

## Quality Gates

Before merge, code must pass:

- [ ] All unit tests pass (`pytest tests/unit/`)
- [ ] All integration tests pass (`pytest tests/integration/`)
- [ ] Coverage ≥ 80% on changed files
- [ ] `ruff check` — zero linting errors
- [ ] `ruff format --check` — code formatted
- [ ] `mypy --strict` — zero type errors on changed files
- [ ] No new `# type: ignore` without justification comment
- [ ] Alembic migration is reversible (`alembic downgrade` works)
- [ ] API schema changes are backward-compatible or versioned
- [ ] No hardcoded secrets or environment-specific values
- [ ] PR description includes context, testing notes, and screenshots if applicable

---

## Open Questions

- [ ] Payment/escrow architecture (Stripe Connect vs. custom escrow — affects data model and async flow)
- [ ] Geographic search implementation (PostGIS vs. Elasticsearch geo queries — affects where spatial logic lives)
- [ ] Event-driven architecture for cross-service notifications (Celery tasks vs. dedicated event bus for scale)
