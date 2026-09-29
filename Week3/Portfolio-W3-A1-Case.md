# Case Study: Connecting CRUD to SQLite & Data Persistence (W3 · A1)
**Bharadwaj Mittapelli** · CSE, Institute of Aeronautical Engineering  
**Live GitHub Repo:** [https://github.com/BharadwajMittapelli/w2-a1-task-crud-api](https://github.com/BharadwajMittapelli/w2-a1-task-crud-api)

---

## 1. The Problem
In Week 2, our task CRUD API used an in-memory JavaScript array. Whenever the server restarted or deployed, every task vanished. Beginners often confuse this behavior with a bug or immediately jump to heavy ORMs and remote database servers before grasping the core architectural boundary: **APIs describe what the system does; databases describe where the system persists state.**

The engineering objective for Week 3: replace the in-memory array with a durable, embedded **SQLite database (`tasks.db`)** without altering a single external API endpoint, URL route, request payload, or response contract.

Key traps to avoid:
- Leaking database-specific details into client responses (e.g. returning SQLite integer `1`/`0` instead of JSON boolean `true`/`false`).
- Race conditions or duplicate seed insertions on consecutive server boots.
- Unhandled SQL syntax errors or crashing the Express process when input is invalid.
- Tying business logic directly to table storage instead of keeping clean query abstraction.

---

## 2. What I Did & Decided

### Embedded SQLite & Engine Selection
- Selected **`better-sqlite3`** over `sqlite3` for synchronous execution, lower overhead, and zero asynchronous callback nesting in high-throughput Express routes.
- Configured **WAL (Write-Ahead Logging)** mode via `PRAGMA journal_mode = WAL` to allow concurrent readers without blocking writes.

### Idempotent Database Initialization (`src/db.js`)
- Handled schema creation automatically:
  ```sql
  CREATE TABLE IF NOT EXISTS tasks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    done INTEGER NOT NULL DEFAULT 0,
    created_at TEXT DEFAULT (datetime('now')),
    updated_at TEXT DEFAULT (datetime('now'))
  );
  ```
- Guarded seed data: checked `SELECT COUNT(*) FROM tasks` before executing seed transaction, ensuring the 3 default tasks are inserted exactly once across infinite server restarts.

### Prepared SQL CRUD Operations (`src/index.js`)
- `GET /tasks`: Executes parameterized `SELECT * FROM tasks` with dynamic SQL clauses for filtering (`WHERE done = ?`), case-insensitive search (`WHERE title LIKE ?`), alphabetical sorting (`ORDER BY title ASC`), and pagination (`LIMIT ? OFFSET ?`).
- `GET /tasks/:id`: Parameterized query `SELECT * FROM tasks WHERE id = ?`, returning `404 Not Found` if row is missing.
- `POST /tasks`: Validates string presence and whitespace, inserts row via `INSERT INTO tasks (title, done) VALUES (?, 0)`, retrieves the record by `lastInsertRowid`, and returns `201 Created`.
- `PUT /tasks/:id`: Validates body presence and types, updates row and sets `updated_at = datetime('now')`, returning `200 OK`.
- `DELETE /tasks/:id`: Checks existence, executes `DELETE FROM tasks WHERE id = ?`, and returns `204 No Content` with an empty body.
- Output mapping: mapped SQLite integer flags (`0`/`1`) to JavaScript booleans (`Boolean(row.done)`), preserving 100% backward compatibility with API clients.

### Stage 4 SQL Exploration (`scripts/explore-sql.js`)
- Wrote and executed manual SQL queries for listing, filtering, counting (`SELECT COUNT(*)`), bulk updates, and deletions, verifying database state synchronization.

---

## 3. What Came of It

- **18/18 Automated Tests Passing (`npm test`)**: Including simulated server shutdown and reboot verifying tasks persist cleanly on disk.
- **Zero Client-Side Breaking Changes**: Swagger UI at `/docs` and existing client integrations continue to function identically with zero code modifications.
- **Auto-Bootstrapping**: New clones running `npm start` automatically create `tasks.db`, tables, and seed rows without manual migration scripts.
- **Clean Git Progression**: All 6 required stages (`Stage 0` through `Stage 5`) committed with exact stage messages and pushed to GitHub.

---

*If you'd like to discuss backend architecture, status code contracts, or agent boundaries, email me to book a 15-minute system review.*
