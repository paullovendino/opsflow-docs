# OpsFlow Coding Standards

## General

- Follow Laravel 13 best practices.
- Follow PSR-12 coding standards.
- Keep controllers thin.
- Move business logic into Services.
- Use Form Requests for validation.
- Use API Resources for responses.
- Prefer Eloquent relationships over raw queries.
- Use dependency injection.
- Use strict typing where possible (`declare(strict_types=1)`).
- Use PHP Enums instead of magic strings.
- Avoid duplicated code.
- Do not introduce packages unless approved.

---

## Backend Structure

```text
app/
├── Actions/
├── Enums/
├── Exceptions/
├── Helpers/
├── Http/
│   ├── Controllers/
│   │   └── Api/
│   │       ├── BaseApiController.php
│   │       └── V1/
│   ├── Middleware/
│   ├── Requests/
│   └── Resources/
├── Models/
├── Policies/
├── Providers/
├── Repositories/
├── Services/
├── Traits/
```

---

## API

- Prefix all routes with `/api/v1`.
- Return the approved JSON response envelope (see below).
- Never return raw models.
- Always use API Resources.
- Use proper HTTP status codes.
- Version controllers under `App\Http\Controllers\Api\V1`.

### Response Envelope

All API responses must use this structure:

```json
{
  "success": true,
  "message": "Request completed successfully.",
  "data": {},
  "errors": null,
  "meta": null
}
```

Rules:

- Success responses set `success` to `true` and `errors` to `null`.
- Error responses set `success` to `false` and place details in `errors` when applicable.
- Validation failures use HTTP `422` with field errors in `errors`.
- Pagination puts records in `data` and page info in `meta` (`current_page`, `last_page`, `per_page`, `total`, `from`, `to`).
- Use `App\Traits\ApiResponse` via `BaseApiController`.

---

## Database

- PostgreSQL
- JSONB where appropriate
- Soft Deletes when appropriate
- Foreign keys
- Proper indexes
- Laravel Enums
- Morph relationships where applicable
- Register morph aliases only for existing models via `Relation::enforceMorphMap`
- Add morph aliases incrementally as models are introduced

---

## Authentication

- Laravel Sanctum SPA cookie authentication
- Stateful domains and CORS must remain environment-driven
- Do not invent auth flows that conflict with `decisions/Authentication.md`
- Keep auth business logic in `App\Services\Auth\AuthenticationService`
- Validate login with Form Requests; respond with API Resources
- Protect authenticated routes with `auth:sanctum`
- Keep login guest-only and rate-limited (`guest`, `throttle:login`)
- Pass only `email` and `password` into `Auth::attempt()`

Details: [AUTHENTICATION.md](AUTHENTICATION.md)

---

## Frontend

- Vue 3 Composition API only.
- Use TypeScript.
- Use Pinia.
- Use Axios (`withCredentials: true` for Sanctum SPA auth).
- Use Vue Router guards.
- Keep components small and reusable.

---

## Git

One feature = One commit.

Use descriptive commit messages.
