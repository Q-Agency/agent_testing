# Project Constitution

## Metadata

| Field | Value |
|-------|-------|
| Project Name | Markdown Note App |
| Project Type | GREENFIELD |
| Status | APPROVED |
| Author | Tech Lead / AI-assisted |
| Date | 2026-02-23 |

---

## Overview

A full-stack web application for creating, organizing, and full-text searching markdown notes with folder hierarchy. Users can manage notes in nested folders, write in markdown with live preview, and instantly find content across all notes.

---

## Tech Stack

### Backend

| Layer | Technology | Version |
|-------|------------|---------|
| Language | Python | 3.12 |
| Framework | FastAPI | 0.115.x |
| Database | SQLite | 3.x (stdlib) |
| Testing | pytest | 8.x |
| Linting | Ruff | 0.9.x |
| Type Checking | mypy | 1.14.x |
| Package Manager | pip | (bundled with Python 3.12) |

### Frontend

| Layer | Technology | Version |
|-------|------------|---------|
| Language | TypeScript | 5.7 |
| Framework | React | 19 |
| Build Tool | Vite | 6.x |
| Testing | Vitest | 3.x |
| Styling | TailwindCSS | 4.x |
| Package Manager | npm | (bundled with Node.js) |

---

## Commands

### Backend

| Action | Command |
|--------|---------|
| Run all tests | `pytest` |
| Run single test file | `pytest <file>` |
| Run tests with coverage | `pytest --cov=app --cov-report=term-missing` |
| Build | N/A |
| Lint | `ruff check .` |
| Format | `ruff format .` |
| Type check | `mypy .` |

### Frontend

| Action | Command |
|--------|---------|
| Run all tests | `npx vitest run` |
| Run single test file | `npx vitest run <file>` |
| Run tests with coverage | `npx vitest run --coverage` |
| Build | `npm run build` |
| Lint | `npx eslint .` |
| Type check | `npx tsc --noEmit` |
| Dev server | `npm run dev` |

---

## Coding Standards

### Naming Conventions

#### Backend (Python)

| Element | Convention | Example |
|---------|------------|---------|
| Files | snake_case | `note_service.py` |
| Functions | snake_case | `get_note_by_id()` |
| Classes | PascalCase | `NoteRepository` |
| Variables | snake_case | `folder_path` |
| Constants | UPPER_SNAKE_CASE | `MAX_NOTE_SIZE` |

#### Frontend (TypeScript / React)

| Element | Convention | Example |
|---------|------------|---------|
| Component files | PascalCase | `NoteEditor.tsx` |
| Utility files | camelCase | `noteUtils.ts` |
| Functions | camelCase | `fetchNotes()` |
| Components | PascalCase | `NoteEditor` |
| Variables | camelCase | `selectedNote` |
| Constants | UPPER_SNAKE_CASE | `API_BASE_URL` |
| Types/Interfaces | PascalCase | `NoteResponse` |
| Hooks | camelCase with `use` prefix | `useNotes()` |

### File Structure

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── database.py
│   ├── routers/
│   │   ├── __init__.py
│   │   ├── notes.py
│   │   └── folders.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── note.py
│   │   └── folder.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── note.py
│   │   └── folder.py
│   ├── repositories/
│   │   ├── __init__.py
│   │   ├── note_repository.py
│   │   └── folder_repository.py
│   └── services/
│       ├── __init__.py
│       ├── note_service.py
│       └── folder_service.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_notes.py
│   └── test_folders.py
├── requirements.txt
└── pyproject.toml

frontend/
├── src/
│   ├── main.tsx
│   ├── App.tsx
│   ├── components/
│   ├── pages/
│   ├── hooks/
│   ├── services/
│   ├── types/
│   └── utils/
├── tests/
├── index.html
├── package.json
├── tsconfig.json
├── vite.config.ts
└── tailwind.config.ts
```

### Patterns to Use

- Repository pattern for data access (all DB queries go through repository classes)
- Pydantic schemas for all request/response validation
- FastAPI dependency injection (`Depends`) for shared dependencies

### Patterns to Avoid

- God objects / fat controllers (routes must not contain business logic; delegate to services)
- Raw SQL string concatenation (always use parameterized queries)

---

## Error Handling

| Concern | Approach |
|---------|----------|
| Strategy | Custom exception classes (`raise` in Python, `throw` in TypeScript). Backend exceptions inherit from a base `AppError` class. |
| Logging | No logging — POC project. |
| User-facing errors | RFC 7807 Problem Details JSON (`type`, `title`, `status`, `detail`) |

---

## Testing Standards

| Requirement | Value |
|-------------|-------|
| Minimum Coverage | 80% |
| Unit Tests Required | YES |
| Integration Tests Required | YES |

### Test File Conventions

- **Backend**: Files named `test_*.py` in `backend/tests/`, using pytest fixtures in `conftest.py`
- **Frontend**: Files named `*.test.ts` or `*.test.tsx` colocated with source or in `frontend/tests/`
- Each public function/endpoint must have at least one test
- Use pytest fixtures for database setup/teardown
- Use Vitest mocks for API calls in frontend tests

---

## API Standards

| Convention | Value |
|------------|-------|
| Style | REST |
| Versioning | URL prefix (`/api/v1/`) |
| Error Format | RFC 7807 Problem Details |
| Content Type | `application/json` |

---

## Security Standards

| Concern | Approach |
|---------|----------|
| Input Validation | Pydantic models for all API inputs (backend); TypeScript types for frontend form validation |
| Authentication | Session-based authentication |
| Secrets Handling | Pydantic `BaseSettings` with `.env` file support, `.env` in `.gitignore`, never committed |

---

## Quality Gates

Before merge, code must pass:

- [ ] All backend tests pass (`pytest`)
- [ ] All frontend tests pass (`npx vitest run`)
- [ ] Backend lint clean (`ruff check .`)
- [ ] Backend formatting clean (`ruff format --check .`)
- [ ] Backend type check clean (`mypy .`)
- [ ] Frontend lint clean (`npx eslint .`)
- [ ] Frontend type check clean (`npx tsc --noEmit`)
- [ ] Test coverage >= 80%
