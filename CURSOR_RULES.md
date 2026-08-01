# Cursor Development Rules

## General

- Follow Laravel 12 best practices.
- Follow Vue 3 Composition API.
- Use TypeScript.
- Write clean, readable code.
- Avoid duplicated logic.
- Prefer dependency injection.
- Do not generate unnecessary comments.
- Keep controllers thin.
- Put business logic in services when appropriate.
- Use Form Requests for validation.
- Use API Resources for responses.
- Use Eloquent relationships instead of manual joins when appropriate.
- Follow PSR-12 coding standards.
- Do not introduce packages unless requested.

## API

- All endpoints must be under `/api/v1`.
- Return consistent JSON responses.
- Use proper HTTP status codes.

## Frontend

- Use Pinia for global state.
- Use Axios for API requests.
- Use Vue Router navigation guards.
- Use Composition API only.