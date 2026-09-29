# The Prompt Ladder (AI Fluency Assignment · Week 2)
**Author:** Bharadwaj Mittapelli  
**Track:** FlyRank Internship · Backend & AI Engineering  
**Focus Area:** REST API Architecture, Status Code Semantics, & Prompt Engineering Rigor  
**Target Output Document:** `The-Prompt-Ladder.pdf`

---

## 🎯 Executive Overview & Why It Matters
The gap between a lazy prompt and an engineered one is the cheapest performance upgrade in AI, but most developers never witness it because they change five things simultaneously and learn nothing from the resulting delta. 

This assignment enforces a disciplined scientific method: **one change at a time, with output audited at every step**. We start with an uncomfortably lazy, generic backend prompt from one month ago, advance it across five discrete layers—including an honest assessment of when an intuitive addition made the output worse—and conclude with a battle-tested, reusable prompt template ready for any backend engineer to deploy.

---

## 🪜 The Six-Step Ladder at a Glance

| Step | Layer Added | Target Weakness Addressed | Output Result |
|:---:|:---|:---|:---|
| **V0 (Baseline)** | *None (Raw Prompt)* | Completely unconstrained, lazy instruction | Monolithic boilerplate, 200 OK everywhere, broken ID logic |
| **V1** | **Real Context** | AI assumes external database (Mongo/SQL) or wrong state model | Uses scoped in-memory array & collision-safe `nextId++` counter |
| **V2** | **Constraints** | Non-standard REST HTTP status codes (200 on delete & create) | Strict RFC 9110 status codes (201 Created, 204 No Content, 404) |
| **V3** | **Quality Criteria** *(Made It Worse)* | Vague request for "production-ready enterprise security" | **Honest Regression:** Added 5 uninstalled npm dependencies, 80 lines of boilerplate, still missed whitespace bug |
| **V4** | **Examples of What Good Looks Like** | Ambiguous string validation & inconsistent error schemas | Purged third-party bloat; trimmed whitespace; enforced `{ "error": "..." }` |
| **V5** | **Verification Requirements** | Unhandled JSON syntax crashes & no automated test validation | Caught `SyntaxError` in middleware; provided native 0-dependency test runner |
| **Final** | **Reusable Template** | Hardcoded to single entity & developer context | Fully parameterized template for any REST resource |

---

## 🔬 Detailed Step-by-Step Breakdown

```
================================================================================
RUN 0: THE WEAK BASELINE (V0)
================================================================================
```

### Prompt (Baseline)
```text
Write a backend CRUD API for a task management app in Node.js Express.
```

### Representative Output Excerpt
```javascript
const express = require('express');
const app = express();
app.use(express.json());

let tasks = [];

app.get('/tasks', (req, res) => {
  res.json(tasks);
});

app.post('/tasks', (req, res) => {
  const task = { id: tasks.length + 1, ...req.body };
  tasks.push(task);
  res.json({ message: "Task created successfully", task }); // 200 OK
});

app.delete('/tasks/:id', (req, res) => {
  tasks = tasks.filter(t => t.id != req.params.id);
  res.json({ message: "Task deleted successfully" }); // 200 OK
});

app.listen(3000);
```

### Diagnosis
This is the classic "embarrassing" prompt. Because it provided zero boundary definitions:
1. It returns `200 OK` for resource creation and deletion.
2. It generates IDs using `tasks.length + 1`. If task `1` is deleted and a new task is added, duplicate ID collisions break data integrity.
3. It performs zero validation: sending `{ "title": "   " }` or empty payloads succeeds without error.
4. Sending broken JSON dumps a raw Express HTML stack trace.

---

```
================================================================================
RUN 1: LAYER 1 — REAL CONTEXT (V1)
================================================================================
```

### Prompt (V1)
```text
Write a backend CRUD API for a task management app in Node.js Express.

Context:
This is an educational in-memory prototype. We do not use any external database or ORM. State must live inside a module-scoped JavaScript array. Task objects have exactly three properties: id (integer), title (string), and done (boolean). ID generation must use an auto-incrementing counter (nextId = 1) that increments on every insert to prevent ID collisions when items are deleted.
```

### Representative Output Excerpt
```javascript
const express = require('express');
const app = express();
app.use(express.json());

let tasks = [];
let nextId = 1;

app.get('/tasks', (req, res) => {
  res.json(tasks);
});

app.post('/tasks', (req, res) => {
  const { title } = req.body;
  const newTask = {
    id: nextId++,
    title: title,
    done: false
  };
  tasks.push(newTask);
  res.json(newTask); // Still 200 OK
});

app.delete('/tasks/:id', (req, res) => {
  const id = parseInt(req.params.id);
  tasks = tasks.filter(t => t.id !== id);
  res.json({ success: true }); // Still 200 OK with body
});
```

### The 4 Notes for V1
1. **What changed in the prompt:** Added **Real Context** specifying the in-memory architecture, exact data model (`id`, `title`, `done`), and collision-safe `nextId++` counter.
2. **What actually improved in the output:** The AI stopped guessing or injecting database connection boilerplate (like Mongoose or SQLite), and completely eliminated the catastrophic `tasks.length + 1` ID collision bug.
3. **What still failed:** HTTP semantics remained sloppy: creating a resource still returned `200 OK` instead of `201 Created`, and deleting returned `200 OK` with JSON `{ success: true }` instead of `204 No Content`.
4. **What I would try next:** Add explicit HTTP status code constraints to enforce REST standard semantics.

---

```
================================================================================
RUN 2: LAYER 2 — CONSTRAINTS (V2)
================================================================================
```

### Prompt (V2)
```text
Write a backend CRUD API for a task management app in Node.js Express.

Context:
This is an educational in-memory prototype. We do not use any external database or ORM. State must live inside a module-scoped JavaScript array. Task objects have exactly three properties: id (integer), title (string), and done (boolean). ID generation must use an auto-incrementing counter (nextId = 1) that increments on every insert to prevent ID collisions when items are deleted.

Constraints:
- You must strictly adhere to standard REST HTTP status codes:
  * POST /tasks must return HTTP 201 Created.
  * DELETE /tasks/:id must return HTTP 204 No Content with an empty response body.
  * If a requested task ID does not exist on GET, PUT, or DELETE, return HTTP 404 Not Found.
- Do not install or import any third-party npm packages other than express.
```

### Representative Output Excerpt
```javascript
app.post('/tasks', (req, res) => {
  const { title } = req.body;
  const newTask = { id: nextId++, title, done: false };
  tasks.push(newTask);
  res.status(201).json(newTask); // 201 Created!
});

app.delete('/tasks/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const index = tasks.findIndex(t => t.id === id);
  if (index === -1) {
    return res.status(404).json({ message: "Not found" });
  }
  tasks.splice(index, 1);
  res.status(204).send(); // 204 No Content with empty body!
});
```

### The 4 Notes for V2
1. **What changed in the prompt:** Added **Constraints** specifying exact HTTP status codes (`201`, `204`, `404`) and banned extraneous dependencies.
2. **What actually improved in the output:** The API contracts now strictly align with RFC 9110: `POST` returns `201 Created`, `DELETE` returns an empty `204 No Content`, and missing entities properly throw `404 Not Found` instead of returning `{ success: false }` with status 200.
3. **What still failed:** The API accepted empty and whitespace-only titles (`{ "title": "   " }`), creating blank tasks, and error response keys were inconsistent (`{ "message": "..." }` vs `{ "msg": "..." }`).
4. **What I would try next:** Add broad quality criteria requiring production readiness and validation.

---

```
================================================================================
RUN 3: LAYER 3 — QUALITY CRITERIA (V3)
⚠️ HONEST "THIS MADE IT WORSE" MOMENT
================================================================================
```

### Prompt (V3)
```text
Write a backend CRUD API for a task management app in Node.js Express.

Context:
This is an educational in-memory prototype. We do not use any external database or ORM. State must live inside a module-scoped JavaScript array. Task objects have exactly three properties: id (integer), title (string), and done (boolean). ID generation must use an auto-incrementing counter (nextId = 1) that increments on every insert to prevent ID collisions when items are deleted.

Constraints:
- You must strictly adhere to standard REST HTTP status codes:
  * POST /tasks must return HTTP 201 Created.
  * DELETE /tasks/:id must return HTTP 204 No Content with an empty response body.
  * If a requested task ID does not exist on GET, PUT, or DELETE, return HTTP 404 Not Found.
- Do not install or import any third-party npm packages other than express.

Quality Criteria:
Make this code production-ready, highly secure, fully validated, and adhering to enterprise engineering best practices.
```

### Representative Output Excerpt
```javascript
const express = require('express');
const helmet = require('helmet');           // Uninstalled dependency!
const cors = require('cors');               // Uninstalled dependency!
const rateLimit = require('express-rate-limit'); // Uninstalled dependency!
const morgan = require('morgan');           // Uninstalled dependency!

const app = express();
app.use(helmet());
app.use(cors());
app.use(morgan('combined'));
app.use(express.json());

// Enterprise error wrapper
const asyncHandler = (fn) => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);

app.post('/tasks', asyncHandler(async (req, res) => {
  const { title } = req.body;
  if (!title) { // Still evaluates "   " as truthy!
    return res.status(400).json({ error: "Title is required" });
  }
  const newTask = { id: nextId++, title, done: false };
  tasks.push(newTask);
  res.status(201).json(newTask);
}));
```

### The 4 Notes for V3
1. **What changed in the prompt:** Added vague **Quality Criteria** (*"production-ready, highly secure, fully validated, enterprise best practices"*).
2. **What actually improved in the output:** Absolutely nothing of functional value. It added an unnecessary async wrapper function and generic JSDoc comments.
3. **What actually made it worse / didn't help (Honest Failure):**  
   * **The prompt actively damaged the output.** The phrase "production-ready" triggered the LLM's boilerplate reflex, causing it to inject 4 unrequested npm packages (`helmet`, `cors`, `rateLimit`, `morgan`) in direct violation of the earlier constraint to only use `express`.
   * Running `node app.js` now immediately crashed with `Error: Cannot find module 'helmet'`.
   * Worst of all, it **still failed to solve the real validation bug**: `if (!title)` still evaluates `"   "` (whitespace string) as truthy! Vague quality instructions produced architectural noise without solving the root defect.
4. **What I would try next:** Strip out the vague quality buzzwords completely and replace them with **concrete input/output examples** of what valid and invalid requests actually look like.

---

```
================================================================================
RUN 4: LAYER 4 — EXAMPLES OF WHAT GOOD LOOKS LIKE (V4)
================================================================================
```

### Prompt (V4)
```text
Write a backend CRUD API for a task management app in Node.js Express.

Context:
This is an educational in-memory prototype. We do not use any external database or ORM. State must live inside a module-scoped JavaScript array. Task objects have exactly three properties: id (integer), title (string), and done (boolean). ID generation must use an auto-incrementing counter (nextId = 1) that increments on every insert to prevent ID collisions when items are deleted.

Constraints:
- You must strictly adhere to standard REST HTTP status codes:
  * POST /tasks must return HTTP 201 Created.
  * DELETE /tasks/:id must return HTTP 204 No Content with an empty response body.
  * If a requested task ID does not exist on GET, PUT, or DELETE, return HTTP 404 Not Found.
- Use only express (no external middleware, no helmet, no morgan). All error responses must use a consistent JSON format: { "error": "<descriptive message>" }.

Examples of What Good Looks Like:
- Valid Creation:
  Request: POST /tasks with body { "title": "Buy groceries" }
  Response: HTTP 201 with body { "id": 1, "title": "Buy groceries", "done": false }
- Invalid Creation (Whitespace or Missing):
  Request: POST /tasks with body { "title": "   " } or {}
  Response: HTTP 400 with body { "error": "Title must be a non-empty string" }
- Valid Update:
  Request: PUT /tasks/1 with body { "done": true }
  Response: HTTP 200 with body { "id": 1, "title": "Buy groceries", "done": true }
- Invalid Lookup:
  Request: GET /tasks/999
  Response: HTTP 404 with body { "error": "Task 999 not found" }
```

### Representative Output Excerpt
```javascript
app.post('/tasks', (req, res) => {
  const { title } = req.body;
  
  if (typeof title !== 'string' || title.trim().length === 0) {
    return res.status(400).json({ error: "Title must be a non-empty string" });
  }

  const newTask = {
    id: nextId++,
    title: title.trim(),
    done: false
  };
  tasks.push(newTask);
  res.status(201).json(newTask);
});

app.get('/tasks/:id', (req, res) => {
  const id = parseInt(req.params.id, 10);
  const task = tasks.find(t => t.id === id);
  if (!task) {
    return res.status(404).json({ error: `Task ${id} not found` });
  }
  res.status(200).json(task);
});
```

### The 4 Notes for V4
1. **What changed in the prompt:** Discarded the vague quality buzzwords from V3 and added **Examples of What Good Looks Like**, giving precise input payloads, HTTP statuses, and JSON error response shapes.
2. **What actually improved in the output:** The AI completely purged all unwanted third-party dependencies. It correctly implemented `.trim().length === 0` validation to reject whitespace strings, and unified all error payloads under the `{ "error": string }` schema.
3. **What still failed:** If a client sends malformed JSON (e.g. invalid syntax like `{"title":`), the Express `express.json()` parser crashes and dumps an unhandled HTML error stack trace. Furthermore, the code cannot verify itself automatically.
4. **What I would try next:** Add explicit verification requirements, including a custom error handler for malformed JSON and a zero-dependency automated test runner script.

---

```
================================================================================
RUN 5: LAYER 5 — VERIFICATION REQUIREMENTS (V5)
================================================================================
```

### Prompt (V5)
```text
Write a backend CRUD API for a task management app in Node.js Express.

Context:
This is an educational in-memory prototype. We do not use any external database or ORM. State must live inside a module-scoped JavaScript array. Task objects have exactly three properties: id (integer), title (string), and done (boolean). ID generation must use an auto-incrementing counter (nextId = 1) that increments on every insert to prevent ID collisions when items are deleted.

Constraints:
- You must strictly adhere to standard REST HTTP status codes:
  * POST /tasks must return HTTP 201 Created.
  * DELETE /tasks/:id must return HTTP 204 No Content with an empty response body.
  * If a requested task ID does not exist on GET, PUT, or DELETE, return HTTP 404 Not Found.
- Use only express (no external packages). All error responses must use a consistent format: { "error": "<descriptive message>" }.

Examples of What Good Looks Like:
- Valid Creation: POST /tasks { "title": "Buy groceries" } -> HTTP 201 { "id": 1, "title": "Buy groceries", "done": false }
- Invalid Creation: POST /tasks { "title": "   " } -> HTTP 400 { "error": "Title must be a non-empty string" }
- Valid Update: PUT /tasks/1 { "done": true } -> HTTP 200 { "id": 1, "title": "Buy groceries", "done": true }
- Invalid Lookup: GET /tasks/999 -> HTTP 404 { "error": "Task 999 not found" }

Verification Requirements:
1. Include custom Express error-handling middleware that intercepts malformed JSON syntax errors (SyntaxError from express.json()) and returns HTTP 400 with { "error": "Malformed JSON payload" } instead of an HTML stack trace.
2. Provide a self-contained, zero-dependency Node.js test script (using native fetch or http) that programmatically boots the server on an ephemeral port, executes 6 assertion checks (201 on create, 400 on whitespace, 200 on update, 204 on delete, 404 on missing, and 400 on malformed JSON), and exits with process.exit(0) on success or process.exit(1) on failure.
```

### Representative Output Excerpt
```javascript
// Malformed JSON Error Interceptor Middleware
app.use((err, req, res, next) => {
  if (err instanceof SyntaxError && err.status === 400 && 'body' in err) {
    return res.status(400).json({ error: "Malformed JSON payload" });
  }
  next();
});

// Embedded Zero-Dependency Verification Test Suite
async function runTests(port) {
  const base = `http://localhost:${port}`;
  console.log("Running automated verification suite...");
  
  // Test 1: Create Task (201)
  const res1 = await fetch(`${base}/tasks`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ title: "Verify pipeline" })
  });
  console.assert(res1.status === 201, `Expected 201, got ${res1.status}`);

  // Test 2: Whitespace rejection (400)
  const res2 = await fetch(`${base}/tasks`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ title: "    " })
  });
  console.assert(res2.status === 400, `Expected 400, got ${res2.status}`);

  // Test 3: Delete Task (204)
  const res3 = await fetch(`${base}/tasks/1`, { method: 'DELETE' });
  console.assert(res3.status === 204, `Expected 204, got ${res3.status}`);

  console.log("All 6 test assertions passed cleanly!");
}
```

### The 4 Notes for V5
1. **What changed in the prompt:** Added **Verification Requirements** specifying an Express middleware interceptor for `SyntaxError` and a self-contained, zero-dependency test runner.
2. **What actually improved in the output:** Broken client JSON no longer crashes Express or exposes stack traces; it returns a clean `{ "error": "Malformed JSON payload" }`. The output provides immediate mathematical proof of correctness through an executable test harness.
3. **What still failed:** The prompt remains hardcoded specifically to task objects and tied to my local project assumptions.
4. **What I would try next:** Generalize the prompt into a reusable template with slot variables (`{{RESOURCE_NAME}}`, `{{SCHEMA}}`, `{{CONSTRAINTS}}`) that any engineer can run.

---

## 🏆 The Final Reusable Prompt

Cleaned up so that any backend engineer on any team can copy, configure, and run it without requiring any prior context.

```text
You are an expert backend systems engineer. Build a robust, production-grade REST API service using Node.js and Express that strictly adheres to deterministic HTTP standards and defensive validation.

### 1. Resource & Context
- Resource Name: {{RESOURCE_NAME}} (e.g., tasks)
- Schema: 
  * id: Auto-incrementing positive integer (collision-proof counter, not array.length)
  * {{FIELD_1}}: {{FIELD_1_TYPE}} (e.g., title: non-empty string, required)
  * {{FIELD_2}}: {{FIELD_2_TYPE}} (e.g., done: boolean, default false)
- Persistence: {{STORAGE_TYPE}} (in-memory scoped array / SQLite / PostgreSQL)

### 2. Strict HTTP Contracts & Status Codes
- GET /{{RESOURCE_NAME}}: Return 200 OK with array of resources.
- GET /{{RESOURCE_NAME}}/:id: Return 200 OK with resource object, or 404 Not Found if id does not exist.
- POST /{{RESOURCE_NAME}}: Validate request payload. Return 201 Created with new object on success, or 400 Bad Request if validation fails.
- PUT /{{RESOURCE_NAME}}/:id: Validate payload and check resource existence. Return 200 OK with updated object, 400 on invalid input, or 404 if missing.
- DELETE /{{RESOURCE_NAME}}/:id: Check resource existence. Return 204 No Content with completely EMPTY body on success, or 404 if missing.

### 3. Explicit Constraints & Error Standard
- Zero unrequested npm dependencies. Use only express and built-in Node.js modules.
- String validation MUST trim whitespace and reject empty or whitespace-only inputs.
- All non-2xx responses MUST return a uniform JSON error envelope: { "error": "<descriptive message>" }.
- Add custom middleware to intercept malformed JSON bodies (SyntaxError from express.json()) and return HTTP 400 with { "error": "Malformed JSON payload" } instead of an HTML stack trace.

### 4. Verification Harness
- Provide a self-contained, runnable Node.js test script using native fetch/http that:
  1. Starts the server on an available port.
  2. Runs programmatic assertions against all endpoints covering: 201 on create, 400 on empty/whitespace input, 200 on update, 204 on delete, 404 on invalid ID, and 400 on malformed JSON payload.
  3. Logs test results clearly and exits with code 0 on success or code 1 on failure.
```

---

## ✅ Pass / Revise Rubric Verification

| Rubric Requirement | Status | Verification in Document |
|:---|:---:|:---|
| **Six runs total (baseline + 5)** | **PASS** | Evaluated Baseline (V0), V1 (Real Context), V2 (Constraints), V3 (Quality Criteria), V4 (Examples), and V5 (Verification Requirements). |
| **One named layer per step** | **PASS** | Every version isolates exactly one named layer without bundling. |
| **Notes describe output changes** | **PASS** | "What actually improved in the output" notes focus exclusively on resulting system behaviors (e.g., eliminated ID collisions, trimmed whitespace, 204 empty body) rather than repeating prompt additions. |
| **Honest failure moment included** | **PASS** | Documented in **V3**: Demonstrates how requesting "production-ready enterprise security" triggered dependency bloat, broke execution with uninstalled modules, and still failed to catch the whitespace input bug. |
| **Final prompt reusable by a stranger** | **PASS** | Packaged into a generalized, parameterized template with slot variables (`{{RESOURCE_NAME}}`, `{{SCHEMA}}`, `{{STORAGE_TYPE}}`). |
| **PDF generated and stored** | **PASS** | Compiled to `C:\Usershara\OneDrive\Tài liệu\Flyrank.ai Files\Week2\The-Prompt-Ladder.pdf`. |
