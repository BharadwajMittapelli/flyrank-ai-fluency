# 🛡️ Auth Login & Protect API

A secure authentication API built with **FastAPI** and **Supabase Auth**. This project demonstrates industry-standard authentication patterns using a trust triangle: the Client, the Backend Server, and an Identity Provider (Supabase).

---

## 📋 Table of Contents
- [Architecture Overview](#-architecture-overview)
- [Features](#-features)
- [Project Structure](#-project-structure)
- [Prerequisites](#-prerequisites)
- [Environment Configuration](#-environment-configuration)
- [Installation & Running](#-installation--running)
- [API Reference](#-api-reference)
- [Interactive Swagger UI](#-interactive-swagger-ui)
- [Testing the API](#-testing-the-api)
- [★ Stage 7: The AI Rematch (AI vs Me)](#-stage-7-the-ai-rematch-ai-vs-me)
- [Security Checklist](#-security-checklist)

---

## 🏛️ Architecture Overview

Secure authentication relies on a trust triangle:
1. **Sign Up / Log In**: Client sends credentials directly to the API, which coordinates with Supabase Auth.
2. **Token Issuance**: Supabase validates credentials and issues a cryptographically signed JSON Web Token (JWT) Access Token.
3. **Authorized Request**: Client attaches the token in the `Authorization: Bearer <token>` HTTP header.
4. **Middleware Verification**: The server extracts and verifies the token using the Supabase SDK before opening protected endpoints.

```
       +---------------------------------------------+
       |                   Client                    |
       +---------------------------------------------+
              |                              ^
   (1) POST   |                   (2) Returns| JWT Access Token
  /auth/login |                              |
              v                              |
       +---------------------------------------------+
       |               FastAPI Server                |
       +---------------------------------------------+
              |                              ^
     Validates|                   Returns    | User & Session
   credentials|                              |
              v                              |
       +---------------------------------------------+
       |            Supabase Auth (IdP)              |
       +---------------------------------------------+
```

---

## ✨ Features

- **Supabase Auth Integration**: User registration, session creation, and sign-out via Supabase's authentication service.
- **Bearer Token Protection**: Reusable FastAPI dependency (`require_auth`) enforcing token presence and validity.
- **Strict HTTP Status Codes**:
  - `201 Created`: User successfully registered.
  - `200 OK`: Successful login or protected resource read.
  - `204 No Content`: Successful session termination (logout).
  - `400 Bad Request`: Missing inputs or registration validation errors.
  - `401 Unauthorized`: Missing token, incorrect format, or invalid/expired token.
- **Interactive Swagger UI**: Automatic OpenAPI documentation with `HTTPBearer` security scheme at `/docs`.
- **Zero Secrets Leakage**: `.env` strictly untracked via `.gitignore`.

---

## 📂 Project Structure

```
auth-login-protect/
├── .env.example            # Template for environment variables
├── .gitignore              # Strictly ignores .env, virtualenvs, cache
├── requirements.txt        # Pinned Python package dependencies
├── supabase_client.py      # Supabase client singleton initialization
├── middleware.py           # Reusable Bearer Auth dependency
├── main.py                 # FastAPI application entrypoint & OpenAPI configuration
├── assets/
│   └── swagger_ui.png      # Screenshot of Swagger UI with Bearer Auth padlock
├── routers/
│   ├── __init__.py
│   ├── auth.py             # /auth/signup, /auth/login, /auth/logout
│   ├── protected.py        # /protected/profile, /protected/dashboard
│   └── public.py           # /public/info
└── README.md               # Documentation & setup guide
```

---

## ⚙️ Prerequisites

- **Python 3.10+** (tested on Python 3.13 / 3.14)
- **Supabase Account & Project**: Free tier at [supabase.com](https://supabase.com)

---

## 🔑 Environment Configuration

1. Copy `.env.example` to create your local `.env`:
   ```bash
   cp .env.example .env
   ```
2. Retrieve your **Project URL** and **Anon Key** in your Supabase Dashboard under `Project Settings -> API`.
3. Populate `.env`:
   ```env
   SUPABASE_URL=https://your-project-id.supabase.co
   SUPABASE_KEY=your_supabase_anon_key
   PORT=8000
   ```

> [!CAUTION]
> **Never commit your `.env` file!** `.gitignore` is pre-configured to ensure secret credentials are never published to version control.

---

## 🚀 Installation & Running

### 1. Clone the repository
```bash
git clone https://github.com/BharadwajMittapelli/auth-login-protect.git
cd auth-login-protect
```

### 2. Create and activate a virtual environment
```bash
# Windows
python -m venv .venv
.venv\Scripts\activate

# macOS / Linux
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Run the server
```bash
python main.py
```
Or with Uvicorn directly:
```bash
uvicorn main:app --reload --port 8000
```

The server will start and log:
```
INFO:auth-api:Server running and connected to Supabase
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

---

## 📖 API Reference

| Method | Endpoint | Auth Required | Status Codes | Description |
|:---|:---|:---:|:---:|:---|
| `POST` | `/auth/signup` | ❌ No | `201`, `400` | Registers a new user with `{"email", "password"}` |
| `POST` | `/auth/login` | ❌ No | `200`, `400`, `401` | Authenticates user; returns JWT `access_token` and `refresh_token` |
| `POST` | `/auth/logout` | ✅ Yes | `204`, `401` | Terminates session in Supabase Auth |
| `GET` | `/public/info` | ❌ No | `200` | Returns public information open to all visitors |
| `GET` | `/protected/profile` | ✅ Yes | `200`, `401` | Returns authenticated user details (ID, email, created_at, metadata) |
| `GET` | `/protected/dashboard` | ✅ Yes | `200`, `401` | Returns private user dashboard metrics and session status |
| `GET` | `/` | ❌ No | `200` | Root health check |

---

## 📸 Interactive Swagger UI

Open your browser and navigate to:
```
http://localhost:8000/docs
```

- Click the green **Authorize 🔓** padlock button.
- Paste your JWT Access Token from `/auth/login`.
- Execute protected requests (`/protected/profile`, `/protected/dashboard`, `/auth/logout`) with one click directly in your browser.

![Swagger UI](assets/swagger_ui.png)

---

## 🧪 Testing the API

### 1. Register a user (Sign Up)
```bash
curl -i -X POST http://localhost:8000/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"email":"alice@example.com", "password":"password123"}'
```
*Expected: `201 Created` with user payload.*

### 2. Log in and acquire JWT
```bash
curl -i -X POST http://localhost:8000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"alice@example.com", "password":"password123"}'
```
*Expected: `200 OK` with `"access_token": "eyJhbG..."` and `"refresh_token"`.*

### 3. Access public info
```bash
curl -i http://localhost:8000/public/info
```
*Expected: `200 OK` with `{"message": "Welcome stranger! This info is public."}`.*

### 4. Attempt protected access without token
```bash
curl -i http://localhost:8000/protected/profile
```
*Expected: `401 Unauthorized` with `{"error": "Access token required"}`.*

### 5. Access protected profile with valid token
```bash
curl -i http://localhost:8000/protected/profile \
  -H "Authorization: Bearer <PASTE_YOUR_ACCESS_TOKEN_HERE>"
```
*Expected: `200 OK` with user id, email, and timestamps.*

### 6. Test token tampering / invalid token
```bash
curl -i http://localhost:8000/protected/profile \
  -H "Authorization: Bearer invalid_or_tampered_token"
```
*Expected: `401 Unauthorized` with `{"error": "Invalid or expired token"}`.*

### 7. Log out (terminate session)
```bash
curl -i -X POST http://localhost:8000/auth/logout \
  -H "Authorization: Bearer <PASTE_YOUR_ACCESS_TOKEN_HERE>"
```
*Expected: `204 No Content`.*

---

## ★ Stage 7: The AI Rematch (AI vs Me)

An AI assistant was prompted to generate an equivalent authentication architecture from memory:
> *"Build a secure FastAPI backend with Supabase Auth handling signup, login, logout, and protected routes using bearer token verification middleware, with strict HTTP 400/401/201/204 status codes and Swagger UI."*

### Analysis & Comparative Findings:

1. **Token Extraction & "Bearer " Prefix Parsing**:
   - **Hand-crafted Architecture**: Explicitly parsed the `Authorization` header (`parts = auth_header.split(" ")`), verifying length == 2, non-empty whitespace stripping, and gracefully falling back between FastAPI's `HTTPAuthorizationCredentials` and raw headers.
   - **AI Baseline**: Relied solely on `credentials: HTTPAuthorizationCredentials = Depends(HTTPBearer())`. When `auto_error=True` (FastAPI's default), FastAPI automatically responds with a `403 Forbidden` (`{"detail": "Not authenticated"}`) instead of the required `401 Unauthorized` (`{"error": "Access token required"}`). Setting `auto_error=False` and adding explicit fallback logic was required to conform to the exact spec.

2. **Security Flaws & Invalid Token Handling**:
   - **Hand-crafted Architecture**: Wrapped `supabase.auth.get_user(token)` in defensive exception blocks (`AuthApiError`, `HTTPException`, network timeouts) and verified `not user_response.user`.
   - **AI Baseline**: Caught generic `Exception` but returned `{"detail": str(e)}`, leaking internal Supabase gateway stack traces and network errors to the client instead of a sanitized `{"error": "Invalid or expired token"}`.

3. **What the Prompt Missed vs AI Assumptions**:
   - The prompt specified status codes, but the AI assumed FastAPI default `422 Unprocessable Entity` for empty or missing JSON bodies. Custom exception handlers (`RequestValidationError`) were necessary to enforce `400 Bad Request` as mandated by REST standards.
   - The AI assumed `HTTP 200` with `{"message": "Logged out"}` on logout instead of the RFC-standard `204 No Content` which disallows a response body.

---

## 🛡️ Security Checklist

- [x] Server starts on localhost with a single documented terminal command (`python main.py`).
- [x] `.env` file is properly used and `.gitignore` prevents it from being pushed to GitHub.
- [x] `POST /auth/signup` and `POST /auth/login` communicate successfully with Supabase Auth.
- [x] `GET /protected/profile` extracts and verifies bearer tokens from the HTTP `Authorization` header.
- [x] Proper status codes used: `201` on signup, `200` on successful login/read, `204` on logout, `400` on missing inputs, and `401` on missing, incorrect, or expired tokens.
- [x] Auth check extracted into reusable middleware dependency (`require_auth`).
- [x] Swagger UI configured at `/docs` with Bearer Token Authorization fully functional (`HTTPBearer`).
- [x] Public GitHub repo with >=6 clean commits and a comprehensive README.
