# Testing Documentation Guide

## Goal

A standalone `TESTING.md` that goes deeper than the testing sections in `ONBOARDING.md` and `ARCHITECTURE.md`. Those files answer "how do I run the tests?" and "what layers exist?". This file answers "how do I write, extend, and reason about the test suite?"

Do not duplicate what's already in `ONBOARDING.md` (setup commands, single-test invocation) or `ARCHITECTURE.md` (testing approach overview). Link to them instead.

## Sections to include

1. **Strategy** — overall approach: what gets tested at which layer, coverage goals if configured, what is intentionally not tested
2. **Test layers** — one subsection per layer with: what it covers, what it doesn't, when to add a test here vs another layer
3. **Running tests** — full command set including coverage, layer-specific runs, CI-equivalent local run
4. **Writing a test** — concrete patterns per layer, not generic advice
5. **Test data** — fixtures, factories, seeds: what exists and how to add new ones
6. **Mocking** — what gets mocked at each layer, how, and the boundary rule
7. **CI** — what runs where, how to reproduce the CI run locally, how long it takes
8. **Known gaps** — flaky tests, skipped tests, coverage holes — be honest

Omit sections you cannot fill from the code. A short, accurate guide beats a long speculative one.

## Detecting the test setup

**Framework and config:**

| Signal | What it tells you |
|---|---|
| `pytest.ini` / `pyproject.toml [tool.pytest.ini_options]` | pytest config, markers, coverage settings |
| `jest.config.ts` / `jest.config.js` | Jest config, transform rules, coverage thresholds |
| `vitest.config.ts` | Vitest config, include patterns |
| `go test` flags in `Makefile` or CI | Go test strategy, race detection, timeout |
| `cargo test` flags in CI | Rust test strategy |

**Test layers:**

| Signal | Layer |
|---|---|
| `tests/unit/` or `*.test.ts` / `test_*.py` next to source | Unit tests |
| `tests/integration/` or tests importing a DB/HTTP client without mocking | Integration tests |
| `tests/e2e/`, `cypress/`, `playwright/` | End-to-end tests |
| `testcontainers` import | Integration tests spin up real services |
| `hypothesis`, `proptest`, `fast-check` | Property / fuzz tests |

**Test data:**

| Signal | Pattern |
|---|---|
| `conftest.py` with `@pytest.fixture` | pytest fixtures — read them, document the key ones |
| `factory_boy`, `FactoryBot`, `@faker-js/faker` | Factory pattern — document available factories |
| `tests/fixtures/`, `testdata/`, `tests/resources/` | Static fixture files — note what they represent |
| `beforeAll` / `afterAll` with DB setup | Lifecycle-managed test database |

**Coverage:**

| Signal | Tool |
|---|---|
| `pytest-cov` or `--cov` in pytest config | Python coverage via pytest |
| `istanbul` / `nyc` / `c8` in package.json | JS/TS coverage |
| `go test -cover` in CI | Go coverage |
| `.codecov.yml` / `coveralls.yml` | Coverage reporting service |

## Strategy section

Describe what the project tests and why at a high level:
- What percentage of coverage is required (if configured — read the threshold from the config file)
- Whether TDD is the expected workflow (look for a high ratio of tests to source files as a signal)
- What is explicitly left untested and why (look for `# pragma: no cover`, `/* istanbul ignore */`, or skipped test files)

## Test layers section

For each layer, document:
- **Scope**: what it exercises (one function? one service? the full stack?)
- **Dependencies**: what is real vs mocked
- **Speed**: approximate run time if evident from CI logs or test counts
- **When to add here**: the decision rule for a contributor writing a new test

Example format per layer:

~~~markdown
### Unit tests (`tests/unit/`)

Exercise individual functions and classes in isolation. All external calls
(database, HTTP, filesystem) are replaced with mocks or in-memory fakes.

Run: `pytest tests/unit/` (~3 s)

Add a unit test when: the logic has branches, edge cases, or error paths that
are hard to trigger through the full stack.

### Integration tests (`tests/integration/`)

Spin up a real PostgreSQL instance via `testcontainers` and exercise the full
request path from HTTP handler to database. Slower but catches ORM mismatches
and schema drift.

Run: `pytest tests/integration/` (~45 s, requires Docker)
~~~

## Writing a test section

Read 2-3 existing tests at each layer to understand the real pattern before writing this section. Adapt the example to what the codebase actually uses.

~~~markdown
## Writing a unit test

Tests use the `app_client` fixture from `conftest.py`. It provides a test
client backed by an in-memory SQLite database pre-seeded with `tests/fixtures/seed.sql`.

```python
def test_create_item_returns_201(app_client):
    response = app_client.post("/items", json={"name": "widget", "qty": 10})
    assert response.status_code == 201
    assert response.json()["id"] is not None
```

To mock an external HTTP call, use `respx`:

```python
import respx, httpx

def test_webhook_fires_on_create(app_client, respx_mock):
    respx_mock.post("https://hooks.example.com/notify").mock(
        return_value=httpx.Response(200)
    )
    app_client.post("/items", json={"name": "widget", "qty": 10})
    assert respx_mock.calls.call_count == 1
```
~~~

## Mocking section

Document the mocking boundary explicitly — this is the most valuable thing to capture and the hardest to infer without reading the tests:

- What is mocked at the unit layer and how (`unittest.mock`, `jest.mock`, `respx`, `mockk`)
- What hits real infrastructure at the integration layer
- Why the boundary is where it is (cost, speed, reliability)

## CI section

Read `.github/workflows/*.yml` (or equivalent) and document:
- Which test commands run in CI
- Whether layers run in parallel or in sequence
- Any setup steps required before tests (migrations, seed data, service startup)
- How to reproduce the CI run locally in one command

If CI runs something different from `pytest` / `npm test`, document the exact CI command — contributors need to know what the gate actually checks.
