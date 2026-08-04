# Setup / Installation

## Prerequisites

- PHP 8.3+ with extensions: `pdo_pgsql`, `mbstring`, `openssl`, `tokenizer`, `xml`, `ctype`, `json`, `bcmath`
- Composer 2.x
- PostgreSQL 14+ (recommended)
- Node.js 20+ (for frontend later)
- Git

---

## Clone Repositories

```bash
# API
git clone <opsflow-api-url> opsflow-api
cd opsflow-api

# Docs (optional local copy)
git clone <opsflow-docs-url> opsflow-docs
```

---

## Backend (`opsflow-api`)

### 1. Install dependencies

```bash
composer install
```

### 2. Environment

```bash
cp .env.example .env
php artisan key:generate
```

Configure at minimum:

```env
APP_NAME=OpsFlow
APP_URL=http://localhost:8000

DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=opsflow
DB_USERNAME=postgres
DB_PASSWORD=your_password

SESSION_DRIVER=database
SESSION_DOMAIN=localhost
SESSION_SAME_SITE=lax

SANCTUM_STATEFUL_DOMAINS=localhost,localhost:5173,127.0.0.1,127.0.0.1:5173
CORS_ALLOWED_ORIGINS=http://localhost:5173,http://127.0.0.1:5173
FRONTEND_URL=http://localhost:5173
```

### 3. Create database

```sql
CREATE DATABASE opsflow;
```

### 4. Migrate and seed

```bash
php artisan migrate
php artisan db:seed
```

This creates foundation tables (including `roles`, sessions, cache/jobs as configured) and seeds:

- `administrator`
- `project_manager`
- `employee`

### 5. Run the API

```bash
php artisan serve
```

Default: `http://localhost:8000`

### 6. Smoke checks

```bash
curl http://localhost:8000/api/v1/health
```

---

## Authentication Manual Check (Postman / Bruno / Insomnia)

1. Enable cookie jar.
2. `GET http://localhost:8000/sanctum/csrf-cookie`
3. Set header `Origin: http://localhost:5173`
4. Send `X-XSRF-TOKEN` (decoded `XSRF-TOKEN` cookie value) on POST requests.
5. `POST /api/v1/auth/login` with JSON `{ "email": "...", "password": "..." }`
6. `GET /api/v1/auth/me`
7. `POST /api/v1/auth/logout`

Full details: [AUTHENTICATION.md](AUTHENTICATION.md) and [API_SPECIFICATION.md](API_SPECIFICATION.md)

---

## Test Database

Feature tests use PostgreSQL database `opsflow_testing` (see [TESTING.md](TESTING.md)).

```sql
CREATE DATABASE opsflow_testing;
```

Align credentials with `phpunit.xml` or override via environment.

---

## Frontend

`opsflow-web` Milestone 8 Frontend Foundation is implemented (Pinia auth shell).

```sh
cd opsflow-web
npm install
cp .env.example .env
npm run dev
```

Run the SPA on Vite port **5173** so it matches CORS and Sanctum stateful defaults (`http://localhost:5173`). Set `VITE_API_BASE_URL` to the API origin (typically `http://localhost:8000`).

Spec: [docs/MILESTONE_8_FRONTEND_FOUNDATION.md](docs/MILESTONE_8_FRONTEND_FOUNDATION.md)
