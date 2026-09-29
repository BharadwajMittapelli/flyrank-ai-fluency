# Portfolio Case Study — W3·A2: Postgres + Docker

## Overview

**Assignment**: Connect the existing CRUD API to PostgreSQL running in Docker, proving that the layered architecture from earlier assignments allows a storage swap with zero route changes.

**Stack**: Node.js 18 · Express 4 · PostgreSQL 16 · Redis 7 · Docker Compose · Swagger UI

---

## Problem

After W3·A1, the API used SQLite — a solid step from in-memory storage, but still a single-process embedded database. Production systems need a proper client-server database (Postgres), containerized infrastructure (Docker), and environment-based configuration (`.env`).

The challenge: **swap the entire storage backend without touching a single route or service line**.

---

## Approach

### 1. Repository Pattern Payoff

The repository interface established in W3·A1 paid off immediately:

| File Changed | Lines Changed | What Changed |
|---|---|---|
| `src/repositories/postgresTaskRepository.js` | **125 lines (new)** | Full Postgres implementation of the same interface |
| `src/repositories/taskRepository.js` | **3 lines** | Added import + switch logic |
| `src/index.js` | **0 lines** | Nothing. Not a single character. |

This is the architecture proving itself. Routes stayed identical because they only talk to the `taskRepository` interface, never to a specific database.

### 2. Docker Compose — One-Command Stack

```yaml
services:
  db:    postgres:16-alpine   # Auto-runs init.sql via entrypoint
  redis: redis:7-alpine       # Stretch goal: caching layer
  app:   Dockerfile (Node 18) # Waits for db + redis health checks
```

Key decisions:
- **Named volume** (`postgres_data`) for data persistence across `docker compose down`
- **Health-check gating** — app container waits until Postgres passes `pg_isready`
- **`init.sql` auto-execution** via Postgres's `/docker-entrypoint-initdb.d/` convention
- **`.env.example`** committed, `.env` gitignored — secrets never enter version control

### 3. PostgreSQL-Specific SQL

| Feature | SQLite (W3·A1) | PostgreSQL (W3·A2) |
|---|---|---|
| Auto-increment | `INTEGER PRIMARY KEY AUTOINCREMENT` | `SERIAL PRIMARY KEY` |
| Boolean type | Integer (0/1) | Native `BOOLEAN` |
| Case-insensitive search | `LIKE` (case-sensitive by default) | `ILIKE` |
| Timestamps | `TEXT DEFAULT datetime('now')` | `TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP` |
| Reset sequence | Manual `UPDATE sqlite_sequence` | `SELECT setval('tasks_id_seq', ...)` |
| Truncate | `DELETE FROM tasks` | `TRUNCATE TABLE tasks RESTART IDENTITY CASCADE` |

---

## Results

- **`docker compose up`** starts the full stack in ~10 seconds
- All 18 automated tests pass against the Postgres backend
- Data survives `docker compose down` (volume-backed)
- Data is wiped cleanly with `docker compose down -v` (fresh start)
- Swagger UI at `/docs` works identically — the API contract is unchanged
- **Zero route changes** between SQLite and PostgreSQL — architecture validated

---

## Key Takeaways

1. **The Repository Pattern is not over-engineering** — it saved hours of refactoring when swapping databases
2. **Docker Compose is the deployment primitive** — `docker compose up` replaces 5 manual setup steps
3. **Health checks prevent race conditions** — without them, the app crashes trying to connect to a not-yet-ready database
4. **`.env` + `.env.example` is the config standard** — secrets stay local, documentation stays committed
5. **SQL dialects differ** — PostgreSQL's `ILIKE`, `SERIAL`, and `TIMESTAMPTZ` are not SQLite-compatible; the repository pattern isolates these differences
