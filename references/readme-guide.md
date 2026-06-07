# README Generation Guide

## Always include these sections (in this order)

1. **Project name + one-line description** — what it does, not how it works
2. **Quick start** — fewest steps to get something running (install → run → see output)
3. **Usage** — real examples with real commands or code
4. **Configuration** — environment variables, flags, config files
5. **Development setup** — how to run tests, lint, build from source
6. **License** — one line

Only add sections that have content. A README with empty sections is worse than a short one.

## Detect project type to set the right tone

**Library/package** (has `src/` exports, no `main` entry, published to npm/PyPI/crates.io):
- Lead with a code snippet showing the API in use
- Include installation (`npm install`, `pip install`, `cargo add`)
- Document the public API surface with types
- Skip deployment — not applicable

**Application/service** (has a `main` entry, serves HTTP, runs as a process):
- Lead with what problem it solves for the end user
- Include both dev setup AND production deployment
- Show example requests/responses if it's an API
- Link to `DEPLOYMENT.md` if it exists

**CLI tool** (has a `bin` field in package.json, `[[bin]]` in Cargo.toml, or a `[project.scripts]` entry):
- Show `--help` output or command table as the centerpiece
- Provide copy-pasteable example invocations
- Describe input/output formats

**Monorepo** (has `packages/`, `apps/`, or `workspaces`):
- List each package/app with a one-line description
- Show how to run the full suite vs. a single package
- Link to per-package READMEs if they exist

**Data science / notebook** (`notebooks/`, `.ipynb` files, `requirements.txt` with pandas/numpy/torch):
- Lead with what the analysis or model does and on what dataset
- Document how to reproduce results (data download, env setup, notebook order)
- Note any GPU/hardware requirements

**API-only backend** (HTTP server, no frontend, no CLI):
- Lead with the problem the API solves and who calls it
- Show the base URL and an example request/response up front
- Link to full API reference (`docs/api.md`) rather than listing every endpoint in the README

## Concrete examples

### Good one-line description
```
spark-lens — detect performance issues in Apache Spark jobs from the terminal or a web dashboard.
```
Not: "A tool for Apache Spark that uses analyzers to process job data and provide insights."

### Good quick start (application)
```markdown
## Quick start

\`\`\`bash
cp .env.example .env   # fill in DATABASE_URL and SECRET_KEY
docker compose up -d
\`\`\`

Open **http://localhost:8000**. Register an agent:

\`\`\`bash
curl -s -X POST http://localhost:8000/agents/register \
  -H "Content-Type: application/json" \
  -d '{"name": "my-agent"}' | jq .api_key
\`\`\`
```

### Good quick start (library)
```markdown
## Installation

\`\`\`bash
pip install acme-sdk
\`\`\`

\`\`\`python
from acme import Client

client = Client(api_key="...")
result = client.search("example query")
\`\`\`
```

### Good configuration table
```markdown
| Variable       | Required | Default | Description                     |
|----------------|----------|---------|---------------------------------|
| `DATABASE_URL` | yes      | —       | Postgres connection string      |
| `SECRET_KEY`   | yes      | —       | 32-byte random hex string       |
| `LOG_LEVEL`    | no       | `info`  | One of: debug, info, warn, error|
```

## Stack-specific quick start commands

| Stack       | Install                               | Run                      | Test          |
|-------------|---------------------------------------|--------------------------|---------------|
| Node.js     | `npm install`                         | `npm start`              | `npm test`    |
| Python      | `pip install -e .`                    | `python -m <module>`     | `pytest`      |
| Go          | `go mod download`                     | `go run ./cmd/...`       | `go test ./...` |
| Rust        | (none needed)                         | `cargo run`              | `cargo test`  |
| Scala/Maven | `mvn install -DskipTests`             | (library — no run step)  | `mvn test`    |
| Scala/sbt   | `sbt compile`                         | `sbt run`                | `sbt test`    |
| Java/Kotlin | `./mvnw install` or `./gradlew build` | `./mvnw spring-boot:run` | `./mvnw test` |
| Flutter     | `flutter pub get`                     | `flutter run`            | `flutter test`|
| PHP         | `composer install`                    | `php -S localhost:8000`  | `./vendor/bin/phpunit` |
| C#/.NET     | `dotnet restore`                      | `dotnet run`             | `dotnet test` |

Read the actual scripts in `package.json` / `Makefile` / `justfile` — prefer those over generic commands.

## Badges (include only if CI exists)

If `.github/workflows/` is present, you may add a build status badge as the first line. Keep it to one or two — don't badge-spam.

## What to leave out

- Do not describe internal implementation details (that belongs in ARCHITECTURE.md)
- Do not list every file and directory
- Do not explain what Git or Node.js is
- Do not copy-paste code comments verbatim — paraphrase at a higher level
- Do not include a "Contributing" section unless there's a CONTRIBUTING.md to link to
