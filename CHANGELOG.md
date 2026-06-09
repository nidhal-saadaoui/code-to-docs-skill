# Changelog

All notable changes to this project will be documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Added
- `testing` subcommand — generates `TESTING.md` covering test strategy, layers, mocking boundary, test data patterns, coverage config, and CI behavior
- `data` subcommand — generates `docs/data.md` for data models, dbt lineage, relational schemas (ORM/migrations), Avro/Protobuf contracts, and data quality checks
- `glossary` subcommand — generates `GLOSSARY.md` with domain terms extracted from model names, enums, constants, and comments; documents state machines, synonyms, and business rule constants
- `userguide` subcommand — generates `docs/user-guide.md` for end users or operators; always asks audience, primary workflow, and roles before writing; adapts structure to web app, CLI, API, or multi-role product
- Pipeline / big data detection in `architecture-guide.md`: Airflow, Prefect, Dagster, Spark, dbt, Flink, Ray — with `graph LR` pipeline diagram guidance
- Big data cross-cutting signals added to Step 1 (`dbt_project.yml`, `dags/`, `SparkSession`, `*.avsc`, `*.proto`)

---

## [1.0.0] — 2026-06-07

Initial release.

### Added

**Subcommands**
- `readme` — generates `README.md` with stack-specific quick start, config table, and project type detection (library, app, CLI, monorepo, data science, API-only backend)
- `arch` — generates `ARCHITECTURE.md` with Mermaid component diagram, data flow, testing approach, and key decisions sections
- `api` — generates `docs/api.md` from route handlers, inline docs, and schema files; supports REST, GraphQL, Library, and OpenAPI formats
- `onboard` — generates `ONBOARDING.md` with prerequisites, setup commands, single-test commands, and writing-a-test section
- `deployment` — generates `DEPLOYMENT.md` with deployment model detection, env vars table, health check, rollback, and monitoring sections
- `adr` — generates `docs/adr/NNN-title.md` files (one per architectural decision) using the Nygard format; detects surprising choices and asks targeted questions
- `update` — incrementally updates existing docs scoped to `git diff main...HEAD`; maps changed files to affected doc types

**Stack detection** — 10 stacks: React/Next.js, Node.js, Python, Go, Rust, Flutter/Dart, Scala/Maven, Scala/sbt, Java/Kotlin, PHP, C#/.NET

**Output options**
- Custom output directory (e.g. "put it in docs/")
- GitHub Wiki format (`[[Page Name]]` links, flat filenames, no front matter)

**Quality guardrails**
- Decision tree for existing docs: extend, replace, stop-and-ask, or edit depending on current state
- Clarifying questions capped at 3 per session (5 for `adr`), batched in a single message
- Contradiction detection: stops entirely when existing docs conflict with code in multiple major ways

**Evals** — 12 test cases covering all subcommands, the contradiction branch, custom output directory, and GitHub Wiki format

**Preview** — complete example output for all subcommands generated for a fictional Python/FastAPI project
