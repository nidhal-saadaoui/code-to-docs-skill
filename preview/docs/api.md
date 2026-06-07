# API Reference

Base URL: `http://localhost:8080`

## Authentication

None. Beacon is designed for internal network use only. See [ADR-003](adr/003-no-authentication.md) for the rationale.

---

## Services

### POST /services

Register a service for health monitoring.

**Request body**

```json
{
  "name": "payments-api",
  "url": "http://payments:8000/health",
  "interval_seconds": 60
}
```

| Field              | Type    | Required | Description                                    |
|--------------------|---------|----------|------------------------------------------------|
| `name`             | string  | yes      | Human-readable label for this service          |
| `url`              | string  | yes      | URL beacon will GET on each check              |
| `interval_seconds` | integer | no       | Poll interval; defaults to `CHECK_INTERVAL_SECONDS` env var |

**Response**

```json
{
  "id": 1,
  "name": "payments-api",
  "url": "http://payments:8000/health",
  "interval_seconds": 60,
  "created_at": "2026-06-07T10:00:00Z"
}
```

| Code | Meaning                        |
|------|--------------------------------|
| 201  | Service registered             |
| 422  | Validation error (missing field, invalid URL) |
| 409  | A service with this name already exists |

---

### GET /services

List all registered services.

**Response**

```json
[
  {
    "id": 1,
    "name": "payments-api",
    "url": "http://payments:8000/health",
    "interval_seconds": 60,
    "created_at": "2026-06-07T10:00:00Z",
    "last_healthy": true,
    "last_checked_at": "2026-06-07T10:05:00Z"
  }
]
```

`last_healthy` and `last_checked_at` are `null` if no checks have run yet.

---

### GET /services/{id}

Get a single registered service.

**Path parameters**

| Name | Type    | Description    |
|------|---------|----------------|
| `id` | integer | Service ID     |

**Response** — same shape as a single item from `GET /services`.

| Code | Meaning           |
|------|-------------------|
| 200  | Service found     |
| 404  | Service not found |

---

### DELETE /services/{id}

Deregister a service. Stops future polling and removes all check history.

**Response**

```json
{ "deleted": 1 }
```

| Code | Meaning           |
|------|-------------------|
| 200  | Service deleted   |
| 404  | Service not found |

---

## Health history

### GET /services/{id}/health

Returns the check history for a service, most recent first.

**Query parameters**

| Name    | Type    | Required | Default | Description                   |
|---------|---------|----------|---------|-------------------------------|
| `limit` | integer | no       | `100`   | Maximum records to return     |

**Response**

```json
{
  "service_id": 1,
  "name": "payments-api",
  "uptime_pct": 99.2,
  "checks": [
    {
      "id": 412,
      "timestamp": "2026-06-07T10:05:00Z",
      "status_code": 200,
      "latency_ms": 34,
      "healthy": true
    },
    {
      "id": 411,
      "timestamp": "2026-06-07T10:04:00Z",
      "status_code": 503,
      "latency_ms": 5012,
      "healthy": false
    }
  ]
}
```

`uptime_pct` is computed over the returned `checks` window, not all-time. A check is marked `healthy: false` if the status code is not 2xx or the request times out (5 s).

| Code | Meaning           |
|------|-------------------|
| 200  | History returned  |
| 404  | Service not found |

---

## Liveness

### GET /health

Liveness check. Confirms the API is running and the database is reachable.

**Response**

```json
{
  "status": "ok",
  "database": "reachable",
  "registered_services": 3,
  "scheduler_jobs": 3
}
```

`scheduler_jobs` should equal `registered_services` under normal operation. A mismatch indicates the scheduler failed to register a job for a service.
