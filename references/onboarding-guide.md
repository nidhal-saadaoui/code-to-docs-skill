# Onboarding Documentation Guide

## Goal

A new contributor should be able to clone the repo, get it running locally, run the tests, and make a small change — all without asking anyone for help.

## Sections to include (in this order)

1. **Prerequisites** — software that must already be installed with minimum versions
2. **Clone and install** — exact commands from zero to running
3. **Environment setup** — required environment variables and where to get them
4. **Running locally** — how to start the app/service in development mode
5. **Running tests** — how to run the full suite and a single test
6. **Project structure** — what lives where (top-level directories only)
7. **Making a change** — the expected development loop (edit → test → commit)
8. **Common issues** — 3-5 actual failure modes with solutions

Omit sections you can't fill from the code. A shorter, accurate guide beats a long, speculative one.

## Prerequisites detection

Look for minimum version requirements in:
- `package.json` → `engines.node`
- `.nvmrc` / `.node-version` → exact Node version
- `pyproject.toml` → `[tool.poetry.dependencies] python`
- `.python-version` → exact Python version
- `go.mod` → `go` directive
- `rust-toolchain.toml` → `channel`
- `pom.xml` → `java.version` property
- `.tool-versions` (asdf) → all tool versions at once

If no version constraints are found, check CI config (`.github/workflows/*.yml`) for the versions used there.

## Environment variables

Read `.env.example`, `.env.sample`, or `config/` files to find required env vars. For each variable:

```markdown
| Variable          | Required | Example value         | Description                        |
|-------------------|----------|-----------------------|------------------------------------|
| `DATABASE_URL`    | yes      | `postgres://localhost/myapp_dev` | Primary database connection |
| `SECRET_KEY`      | yes      | (generate with `openssl rand -hex 32`) | Session signing key   |
| `STRIPE_API_KEY`  | no       | `sk_test_...`         | Needed for payment features only   |
```

If there's no `.env.example`, read the config loading code (often `config.py`, `config.ts`, `cmd/root.go`) to find what's expected.

## Stack-specific setup commands

**Node.js**
```bash
git clone <repo>
cd <project>
cp .env.example .env   # if present
npm install
npm run dev
```

**Python**
```bash
git clone <repo>
cd <project>
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -e ".[dev]"
cp .env.example .env
python -m pytest
```

**Go**
```bash
git clone <repo>
cd <project>
go mod download
cp .env.example .env
go run ./cmd/server
```

**Rust**
```bash
git clone <repo>
cd <project>
cargo build
cargo test
cargo run
```

**Docker Compose (add this if docker-compose.yml exists)**
```bash
cp .env.example .env
docker compose up -d
```

Use the actual scripts from `Makefile`, `justfile`, or `package.json` scripts if they exist — they often handle setup steps that the above commands miss.

## Running a single test

This is one of the most useful things to document and one of the most often omitted:

| Stack       | Run all tests         | Run one test file                    | Run one test by name                        |
|-------------|----------------------|---------------------------------------|---------------------------------------------|
| Node/Jest   | `npm test`           | `npx jest path/to/file.test.ts`      | `npx jest -t "test name"`                   |
| Vitest      | `npm test -- --run`  | `npx vitest run path/to/file.test.ts` | `npx vitest run -t "test name"`            |
| pytest      | `pytest`             | `pytest tests/test_foo.py`           | `pytest -k "test_name"`                     |
| Go          | `go test ./...`      | `go test ./pkg/foo/...`              | `go test -run TestName ./...`               |
| Rust        | `cargo test`         | (by module: `cargo test foo::`)      | `cargo test test_name`                      |
| JUnit/Maven | `./mvnw test`        | `./mvnw test -Dtest=FooTest`         | `./mvnw test -Dtest=FooTest#testMethod`     |

Read the CI config to confirm — the matrix often reveals the exact commands used.

## Writing a new test

After the single-test table, add a short section explaining how to write a test for this specific project. Detect the pattern from existing tests — don't write generic advice.

**What to look for:**

| Signal | What it tells you |
|---|---|
| `conftest.py` with fixtures | pytest fixture pattern — document the key fixtures new tests should use |
| `beforeAll` / `afterAll` in test files | Setup/teardown lifecycle — note if a DB or server needs to be started |
| `tests/unit/` + `tests/integration/` split | Two-tier strategy — note what belongs in each |
| Mock imports (`unittest.mock`, `jest.mock`, `mockk`) | Mocking boundary — note what gets mocked vs what hits real dependencies |
| `testcontainers` or `docker-compose.test.yml` | Integration tests spin up real services — note the setup command |
| Fixture files in `tests/resources/` or `testdata/` | Document what fixtures exist and what they represent |

**Format for the "writing a test" section:**

<!-- outer fence is ~~~ to avoid a nested-fence rendering issue in this reference file; the ONBOARDING.md you write should use normal ``` fences -->
~~~markdown
## Writing a test

Tests live in `tests/` and mirror the structure of `src/`. Each module has a
corresponding `test_<module>.py`.

Use the `client` fixture from `conftest.py` — it provides a pre-configured
test client with a seeded in-memory database:

```python
def test_something(client):
    response = client.post("/endpoint", json={...})
    assert response.status_code == 200
```

Unit tests mock external HTTP calls with `respx`. Integration tests in
`tests/integration/` use a real database and require `docker compose up -d db`.
~~~

Adapt the example to what the codebase actually uses — read 2-3 test files to understand the real pattern before writing this section. If the project has no tests, note the absence rather than inventing a pattern.

## Project structure section

Only document top-level directories and files that are non-obvious. Skip anything self-evident (`README.md`, `.gitignore`, `node_modules/`).

Example format:
```
src/
  api/       — HTTP route handlers
  domain/    — core business logic, no framework dependencies
  infra/     — database clients, external service adapters
  config/    — environment and application configuration
tests/       — mirrors src/ structure
scripts/     — one-off maintenance and migration scripts
```

## Common issues section

Look for clues about known failure modes in:
- GitHub Issues (if accessible)
- `CONTRIBUTING.md` or existing docs
- Test setup code (often contains workarounds for flaky behavior)
- Docker/environment-specific conditionals in config

If no clues exist, include generic but accurate entries:
- Port already in use → `lsof -i :<port>` to find the process
- Database not running → check `docker compose ps` or the service status
- Missing env var → which one is missing and where to get it

Do not invent fictional issues.
