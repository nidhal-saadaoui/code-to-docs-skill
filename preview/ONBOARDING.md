# Onboarding

Get the project running locally, run the tests, and make a change — without asking anyone for help.

## Prerequisites

| Tool      | Minimum version | Check                  |
|-----------|-----------------|------------------------|
| Python    | 3.11            | `python --version`     |
| Docker    | 24.x            | `docker --version`     |
| Docker Compose | 2.x        | `docker compose version` |

Python version is pinned in `.python-version`. Use [pyenv](https://github.com/pyenv/pyenv) if you need to manage multiple versions.

## Clone and install

```bash
git clone <repo-url>
cd beacon
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -e ".[dev]"
cp .env.example .env
```

## Environment setup

`.env.example` contains all available variables. For local development, the defaults work without changes:

```
DATABASE_PATH=beacon.db
CHECK_INTERVAL_SECONDS=60
LOG_LEVEL=debug
PORT=8080
```

## Running locally

**With Docker Compose** (recommended — matches production):

```bash
docker compose up -d
```

**Without Docker** (faster iteration):

```bash
uvicorn beacon.main:app --reload --port 8080
```

The API is available at `http://localhost:8080`. The `/health` endpoint confirms it is running.

## Running tests

| Command | What it runs |
|---|---|
| `pytest` | All unit tests |
| `pytest tests/integration/` | Integration tests (requires no running services) |
| `pytest tests/test_routes.py` | Single test file |
| `pytest -k "test_register"` | Tests matching a name pattern |

Unit tests use an in-memory SQLite database and `respx` for HTTP mocking — no external services required. Integration tests start a local `uvicorn` instance; they take a few seconds longer.

## Writing a test

Tests live in `tests/` and mirror the structure of `beacon/`. Each module has a corresponding `test_<module>.py`.

Use the `client` fixture from `conftest.py` — it provides an async test client backed by a fresh in-memory database:

```python
async def test_register_service(client):
    response = await client.post("/services", json={
        "name": "my-service",
        "url": "http://example.com/health",
        "interval_seconds": 30,
    })
    assert response.status_code == 201
    assert response.json()["name"] == "my-service"
```

For tests that involve outbound HTTP (scheduler checks), mock with `respx`:

```python
import respx, httpx

async def test_check_records_latency(client, respx_mock):
    respx_mock.get("http://example.com/health").mock(return_value=httpx.Response(200))
    # trigger a check and assert the result
```

## Project structure

```
beacon/
  main.py        — application entry point, lifespan hooks
  routes/        — one file per route group
  scheduler.py   — APScheduler setup and job management
  db/            — database connection pool and raw SQL helpers
tests/
  conftest.py    — shared fixtures (client, seeded DB)
  integration/   — tests that use a real uvicorn instance
```

## Common issues

**Port 8080 already in use**
```bash
lsof -i :8080   # find the process
kill -9 <PID>
```

**`ModuleNotFoundError: No module named 'beacon'`** — the package is not installed in editable mode. Run `pip install -e ".[dev]"` from the repo root with the virtualenv active.

**Scheduler not polling** — check that `CHECK_INTERVAL_SECONDS` is set and that the registered service URL is reachable from inside Docker. Use `docker compose logs beacon` to see scheduler output.

**`sqlite3.OperationalError: database is locked`** — two processes are writing to the same `beacon.db` file. Stop the background server before running integration tests that write to the same path.
