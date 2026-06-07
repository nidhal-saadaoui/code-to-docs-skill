# ADR-002: FastAPI over Flask

**Status**: accepted

## Context

Beacon runs a background scheduler (APScheduler) that polls registered services on a timer. The scheduler and the HTTP server must coexist in the same process — running a separate scheduler process would require inter-process coordination (a shared database lock or a message queue) that is not justified at this scale.

Flask is synchronous (WSGI). Running APScheduler alongside a WSGI server requires either a background thread or an external process, both of which introduce threading edge cases with SQLite's connection model.

## Decision

We will use FastAPI (ASGI) with `uvicorn`.

The scheduler runs on the same asyncio event loop as the HTTP server, started in FastAPI's `lifespan` context manager. Database writes from scheduler jobs and from HTTP handlers share the same `aiosqlite` connection pool without locking conflicts.

## Consequences

- **Easier**: the entire service runs in a single event loop with no threads. Shutdown is clean — the lifespan hook stops the scheduler before the server closes.
- **Harder**: all database access must be async. Synchronous SQLite calls block the event loop and cause latency spikes. Any contributor adding a new route must use `await` for all DB calls.
- **Technical debt**: none identified at current scope.
