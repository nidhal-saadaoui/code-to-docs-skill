# API Documentation Guide

## Inline documentation takes priority

Before reading route/function bodies, check for inline docs — they are the most authoritative source and reduce how much you need to infer:

| Stack        | Inline doc format       | Where to look                        |
|--------------|-------------------------|--------------------------------------|
| TypeScript   | JSDoc `/** ... */`      | Above function/class declarations    |
| Python       | Docstrings `"""..."""`  | First statement inside function/class|
| Go           | `// FuncName ...`       | Line directly above `func`           |
| Rust         | `/// ...`               | Line directly above `fn`/`struct`    |
| Java/Kotlin  | Javadoc `/** ... */`    | Above method/class                   |
| PHP          | PHPDoc `/** ... */`     | Above function/class                 |

If inline docs exist: use their descriptions directly, filling in types and examples from the source. Don't paraphrase unnecessarily — the author already explained it.

If inline docs are **absent** for a public function/endpoint: infer from the implementation, and flag in Step 3.5 if the behavior is non-obvious or the parameter semantics are ambiguous.

## Determine the API surface type first

**REST API** — look for route definitions (`app.get`, `router.post`, `@GetMapping`, `http.HandleFunc`, `#[get(...)]`). Document each endpoint.

**GraphQL** — look for `.graphql` / `.gql` schema files, or a schema definition in code (`buildSchema`, `gql\`...\``). Document types, queries, mutations, subscriptions.

**Library / SDK** — look for public exports (`export function`, `pub fn`, `public`, `module.exports`). Document the callable surface, not internal helpers.

**OpenAPI / Swagger already exists** — if `openapi.yml`, `swagger.yml`, or `openapi.json` is present, read it and generate human-readable docs from it rather than re-reading all source files.

## REST API documentation structure

For each endpoint, document:

```
### METHOD /path

Brief description of what the endpoint does.

**Authentication**: required / optional / none (and what kind)

**Path parameters**

| Name   | Type   | Description          |
|--------|--------|----------------------|
| `id`   | string | Resource identifier  |

**Query parameters**

| Name     | Type    | Required | Default | Description           |
|----------|---------|----------|---------|-----------------------|
| `limit`  | integer | no       | 20      | Max results to return |

**Request body** (if applicable)

\`\`\`json
{
  "name": "string",
  "active": true
}
\`\`\`

**Response**

\`\`\`json
{
  "id": "abc123",
  "name": "string",
  "createdAt": "2026-01-01T00:00:00Z"
}
\`\`\`

**Status codes**

| Code | Meaning              |
|------|----------------------|
| 200  | Success              |
| 400  | Validation error     |
| 401  | Unauthenticated      |
| 404  | Resource not found   |
```

## Library/SDK documentation structure

For each public function/method/class:

```
### functionName(param1, param2)

Brief description.

**Parameters**

| Name     | Type    | Required | Description              |
|----------|---------|----------|--------------------------|
| `param1` | string  | yes      | The thing being processed|
| `param2` | Options | no       | Configuration object     |

**Returns**: `ReturnType` — description of what it returns

**Example**

\`\`\`typescript
const result = functionName("input", { option: true })
\`\`\`

**Throws**: `ErrorClass` — when this condition occurs (if applicable)
```

## Finding the API surface

**Node.js/Express**: read `routes/`, `router.ts`, `app.ts` — look for `.get(`, `.post(`, `.put(`, `.delete(`, `.patch(` calls. If response/request types live in a separate `types.ts`, `models.ts`, or `dto/` directory, read those too — TypeScript interfaces define the actual shapes used in the API contract.

**Python/FastAPI**: look for `@app.get`, `@router.post`, etc. Read the route handlers first, then read any `models.py` or `schemas.py` files separately — Pydantic models define the actual request/response shapes and are often richer than what's visible in the route signature alone.

**Python/Django**: read `urls.py` to find routes, then the corresponding view classes/functions.

**Go**: look for `http.HandleFunc`, `mux.Handle`, or router-specific patterns (`r.GET`, `e.POST`). Read handler functions for parameter extraction.

**Java/Spring**: look for `@RestController`, `@GetMapping`, `@PostMapping`, etc.

**Rust/Axum or Actix**: look for `Router::new().route(...)` or `#[get(...)]` macros.

## What NOT to document

- Internal helpers, private functions, unexported symbols
- Test utilities
- Generated code (migration files, protobuf outputs)
- Deprecated endpoints (mark them `**Deprecated**` with a migration path, don't expand them)

## Authentication section (at the top)

If authentication is present, write a dedicated intro section before listing endpoints:

```markdown
## Authentication

All endpoints except `/health` require a Bearer token in the `Authorization` header:

\`\`\`
Authorization: Bearer <token>
\`\`\`

Tokens are obtained via `POST /auth/login`. They expire after 24 hours.
```

Detect auth by looking for middleware like `passport`, `jwt.verify`, `@RequiresAuth`, `SecurityConfig`, or token validation in request handlers.
