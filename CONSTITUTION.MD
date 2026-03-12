# Project Constitution — Backend

## Metadata

| Field | Value |
|-------|-------|
| Project Name | AutoMarket — Car Selling Platform |
| Project Type | GREENFIELD |
| Status | DRAFT |
| Author | AI Agency OS / BA Agent |
| Date | 2026-03-12 |

---

## Overview

AutoMarket is an online car selling platform that connects private sellers and dealerships with buyers. The backend provides REST APIs for vehicle listing management, user accounts, search and filtering, messaging between parties, payment/escrow coordination, and dealership inventory management. It serves the web frontend and mobile app as a single source of truth.

---

## Tech Stack

| Layer | Technology | Version |
|-------|------------|---------|
| Language | Python | 3.12+ |
| Framework | FastAPI | 0.115+ |
| Database | PostgreSQL | 16+ |
| ORM | SQLAlchemy | 2.0+ (async with asyncpg) |
| Migrations | Alembic | 1.13+ |
| Search | Elasticsearch | 8.x (vehicle search, hybrid dense + BM25) |
| Cache | Redis | 7+ |
| Auth | Supabase Auth (JWT + RLS) | latest |
| Object Storage | Supabase Storage (vehicle images) | latest |
| Task Queue | Celery + Redis broker | 5.4+ |
| Testing | pytest + pytest-asyncio + httpx | latest |
| Linting | Ruff | latest |
| Type Checking | mypy (strict mode) | latest |
| API Docs | Auto-generated OpenAPI 3.1 via FastAPI | — |
| Containerization | Docker + Docker Compose | — |

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
├── main.py                          # FastAPI app factory, lifespan, middleware
├── config.py                        # Pydantic Settings (env-based config)
├── dependencies.py                  # Shared FastAPI Depends (db session, current user, etc.)
├── models/                          # SQLAlchemy ORM models
│   ├── __init__.py
│   ├── user.py
│   ├── vehicle.py
│   ├── offer.py
│   ├── message.py
│   └── dealership.py
├── schemas/                         # Pydantic request/response schemas
│   ├── __init__.py
│   ├── user.py
│   ├── vehicle.py
│   ├── offer.py
│   ├── message.py
│   └── common.py                    # Pagination, error responses, shared types
├── api/
│   ├── __init__.py
│   ├── v1/
│   │   ├── __init__.py
│   │   ├── router.py                # Aggregates all v1 routers
│   │   ├── vehicles.py              # Vehicle CRUD + search endpoints
│   │   ├── users.py                 # User profile, registration
│   │   ├── offers.py                # Buy offers, counter-offers
│   │   ├── messages.py              # Buyer-seller messaging
│   │   ├── dealerships.py           # Dealership management
│   │   ├── media.py                 # Image upload/management
│   │   └── admin.py                 # Admin/moderation endpoints
├── services/                        # Business logic layer (no HTTP, no DB imports)
│   ├── __init__.py
│   ├── vehicle_service.py
│   ├── search_service.py            # Elasticsearch integration
│   ├── offer_service.py
│   ├── message_service.py
│   ├── media_service.py             # Supabase Storage operations
│   ├── notification_service.py      # Email/push notification dispatch
│   └── pricing_service.py           # Price estimation, market comparison
├── repositories/                    # Data access layer (SQLAlchemy queries)
│   ├── __init__.py
│   ├── base.py                      # Generic CRUD repository
│   ├── vehicle_repository.py
│   ├── user_repository.py
│   └── offer_repository.py
├── tasks/                           # Celery async tasks
│   ├── __init__.py
│   ├── image_processing.py          # Resize, watermark, EXIF strip
│   ├── search_indexing.py           # Elasticsearch sync
│   └── notifications.py             # Async email/push sends
├── middleware/
│   ├── __init__.py
│   ├── logging.py                   # Structured request/response logging
│   └── rate_limit.py                # Per-endpoint rate limiting
├── utils/
│   ├── __init__.py
│   ├── pagination.py                # Cursor-based and offset pagination
│   └── validators.py                # VIN validation, phone format, etc.
├── migrations/                      # Alembic migrations
│   ├── env.py
│   └── versions/
tests/
├── conftest.py                      # Fixtures: test DB, async client, factory boy
├── unit/
│   ├── services/
│   └── utils/
├── integration/
│   ├── api/
│   └── repositories/
├── factories/                       # Factory Boy model factories
│   ├── vehicle_factory.py
│   └── user_factory.py
```

### Patterns to Use

- **Repository pattern** — all database access goes through `repositories/`. Services never import SQLAlchemy directly.
- **Service layer** — all business logic in `services/`. Endpoints are thin wrappers that validate input, call a service, return a response.
- **Dependency injection** — use FastAPI `Depends()` for DB sessions, current user, permissions. No global state.
- **Async everywhere** — all endpoints, DB queries (asyncpg), and HTTP calls (httpx) are async. No sync blocking in the event loop.
- **Pydantic v2 models for all I/O** — strict validation on inbound, explicit serialization on outbound. Use `model_config = ConfigDict(from_attributes=True)` for ORM conversion.
- **Explicit error handling** — custom exception classes in `exceptions.py`, caught by FastAPI exception handlers. Never return raw 500s.
- **Structured logging** — JSON logs via `structlog`. Every log line includes `request_id`, `user_id`, `action`.
- **Feature flags** — use environment variables or a simple DB table for feature toggles. No hardcoded conditionals for rollout logic.

### Patterns to Avoid

- **Fat endpoints** — if an endpoint function exceeds ~20 lines, the logic belongs in a service.
- **N+1 queries** — use `selectinload()` / `joinedload()` on relationships. Review all list endpoints for N+1.
- **Raw SQL in services** — all queries go through repositories. Raw SQL only in migrations or one-off scripts.
- **Mutable default arguments** — never `def func(items=[])`. Use `None` + internal initialization.
- **Catching bare `Exception`** — catch specific exceptions. Let unexpected errors propagate to the global handler.
- **Business logic in models** — SQLAlchemy models are data containers. Validation and logic live in services.
- **Hardcoded secrets** — all secrets via environment variables or Supabase vault. Never in code, never in config files committed to git.

---

## Testing Standards

| Requirement | Value |
|-------------|-------|
| Minimum Coverage | 80% (services and repositories) |
| Unit Tests Required | YES |
| Integration Tests Required | YES |

### Test File Conventions

- Test files mirror source structure: `src/services/vehicle_service.py` → `tests/unit/services/test_vehicle_service.py`
- Prefix all test files with `test_`.
- Use `pytest-asyncio` for all async tests. Mark with `@pytest.mark.asyncio`.
- Use `httpx.AsyncClient` for integration tests against the FastAPI app.
- Use Factory Boy for model factories. Never construct test data inline.
- Integration tests use a real PostgreSQL test database (Docker), not SQLite.
- Each test function tests one behavior. Name format: `test_<action>_<condition>_<expected_result>` (e.g., `test_create_vehicle_missing_vin_returns_422`).
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
| Response Envelope | Top-level `data` key for success, `error` key for failures |

### Key Endpoints (illustrative, not exhaustive)

- `POST /api/v1/vehicles` — create listing
- `GET /api/v1/vehicles/{id}` — get listing detail
- `GET /api/v1/vehicles/search` — search with filters (make, model, year, price range, location, mileage)
- `POST /api/v1/vehicles/{id}/offers` — submit buy offer
- `POST /api/v1/media/upload` — upload vehicle images (multipart)
- `GET /api/v1/users/me` — current user profile
- `GET /api/v1/dealerships/{id}/inventory` — dealership vehicle listing

---

## Security Standards

- All endpoints require authentication except: vehicle search, public listing view, registration, login.
- JWT tokens validated via Supabase Auth. Short-lived access tokens (15 min), long-lived refresh tokens.
- Row-Level Security (RLS) enforced at the PostgreSQL layer via Supabase — users can only modify their own listings and offers.
- All user input validated by Pydantic before reaching services. No raw user strings in SQL or Elasticsearch queries.
- Image uploads validated: file type whitelist (JPEG, PNG, WebP), max size 10MB, EXIF metadata stripped before storage.
- VIN numbers validated against checksum algorithm before persisting.
- Rate limiting on auth endpoints (5 attempts/min), search (30 req/min), listing creation (10/hour).
- CORS restricted to known frontend origins. No wildcard `*` in production.
- All inter-service communication over HTTPS. Internal services use private network, no public exposure.
- Secrets managed via environment variables. No secrets in code or Docker images.
- SQL injection prevented by parameterized queries (SQLAlchemy ORM). XSS prevented by not rendering user content server-side.
- Dependency scanning via `pip-audit` in CI.

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
- [ ] No hardcoded secrets, API keys, or environment-specific values
- [ ] PR description includes context, testing notes, and screenshots (if applicable)

---

## Open Questions

- [ ] Payment/escrow provider selection (Stripe Connect vs. custom escrow flow)
- [ ] Vehicle history report integration (Carfax API vs. alternative)
- [ ] Geographic search implementation: PostGIS vs. Elasticsearch geo queries
- [ ] Image CDN strategy: Supabase Storage direct vs. Cloudflare R2 + CDN
- [ ] Push notification provider for real-time alerts (Firebase vs. OneSignal)
