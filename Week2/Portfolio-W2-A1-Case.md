# Case Study: In-Memory Task CRUD API & AI Code Review (W2 · A1)
**Bharadwaj Mittapelli** · CSE, Institute of Aeronautical Engineering  
**Live GitHub Repo:** [https://github.com/BharadwajMittapelli/w2-a1-task-crud-api](https://github.com/BharadwajMittapelli/w2-a1-task-crud-api)

---

## 1. The Problem
Beginners often overthink CRUD APIs or immediately grab an ORM and database before understanding basic HTTP mechanics. The goal here was to build the heartbeat of backend architecture from scratch: a clean, in-memory task management REST API with standard CRUD operations, strict status code semantics, input validation, interactive OpenAPI/Swagger UI docs, and a structured AI rematch review.

The main traps to avoid:
- Returning `200 OK` for missing resources or empty list responses when a resource doesn't exist.
- Relying on `array.length + 1` for ID generation, which leads to duplicate ID collisions when items are deleted.
- Silent fallbacks or returning HTML stack traces on malformed JSON bodies instead of strict JSON error payloads.
- Assuming AI-generated backend code is "ready for production" without checking edge cases, whitespace handling, or status code contracts.

---

## 2. What I Did & Decided

### Architecture & Framework
- Built the server using **Node.js** and **Express**.
- Kept data strictly in memory using a scoped JavaScript collection with auto-incrementing ID counters (`nextId++`) to guarantee unique IDs regardless of deletions.

### HTTP Contracts & Status Codes
- `GET /` & `GET /health`: API metadata and standard health check (`200 OK`).
- `GET /tasks` & `GET /tasks/:id`: List all tasks or get single task by ID. Returns `404 Not Found` with `{ "error": "Task <id> not found" }` if absent.
- `POST /tasks`: Validates `title` (rejects missing, non-string, or whitespace-only strings like `"   "` with `400 Bad Request`). Sets `done: false` and returns `201 Created`.
- `PUT /tasks/:id`: Replaces title and/or done status with validation on types and body presence. Returns `200 OK`, `400 Bad Request`, or `404 Not Found`.
- `DELETE /tasks/:id`: Splices item from array and returns `204 No Content` with an empty response body (or `404 Not Found`).

### Error Handling & Middleware
- Added custom Express middleware to intercept malformed JSON bodies (e.g. broken syntax) and return clean `{ "error": "Invalid JSON payload in request body." }` with status `400` instead of leaking HTML stack traces.

### Extras Built
- **Filtering & Search**: `GET /tasks?done=true|false` and case-insensitive substring search `GET /tasks?search=keyword`.
- **Pagination**: Offset and limit parameters (`?limit=N&offset=N`) to model real-world API scale and prevent memory saturation.
- **Computed Stats**: `GET /stats` returning dynamic aggregates `{ "total": 3, "done": 1, "open": 2 }`.
- **State Reset**: `POST /reset` to reload initial seed tasks for testing and demos.

### Interactive OpenAPI / Swagger UI
- Created a complete **OpenAPI 3.0.3 specification** (`src/openapi.json`) detailing all paths, parameters, schemas, and response codes. Mounted at `/docs` using `swagger-ui-express`.

### Stage 7: AI Rematch Code Review
- Quarantined an AI-generated implementation in `ai-version/` to test prompt engineering against hand-built code.
- **Flaws identified in AI Round 1**:
  1. Returned `200 OK` on `POST` instead of `201 Created`.
  2. Returned `200 OK` with `{ "message": "Deleted" }` on `DELETE` instead of `204 No Content`.
  3. Used `tasks.length + 1` for ID assignment, causing ID collisions after deletions.
  4. Accepted whitespace-only strings (`"   "`).
  5. Inconsistent error key naming (`{ msg: "..." }` vs `{ error: "..." }`).
- Refined the prompt in Round 2 with explicit contract rules, generating a production-grade rematch that matched hand-crafted quality.

---

## 3. What Came of It

- **16/16 Automated Tests Passed**: Verified via a custom HTTP test runner (`npm test`).
- **9 Granular Git Commits**: Built stage-by-stage (`Stage 0` through `Stage 7` plus `Extras`), demonstrating discipline and incremental delivery.
- **Published GitHub Repository**: Live at [https://github.com/BharadwajMittapelli/w2-a1-task-crud-api](https://github.com/BharadwajMittapelli/w2-a1-task-crud-api).
- **Core takeaway on RAM vs DB**: Documented the "Mortality Experiment" in `README.md` — demonstrating why volatile process memory resets on restart and why persistent storage (Week 3) is necessary for production applications.

---

*If you'd like to discuss backend architecture, status code contracts, or agent boundaries, email me to book a 15-minute system review.*
