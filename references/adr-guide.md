# ADR Documentation Guide

## Goal

Each ADR captures one architectural decision: the context that forced the choice, the decision made, and what it implies going forward. The record is permanent — it explains the past, not the present state. It is not a spec or a plan; it is a record of a decision already made.

## Output location

`docs/adr/NNN-kebab-case-title.md` — use the next available number. If `docs/adr/` already exists, read all existing ADRs before writing new ones.

## ADR file format

~~~markdown
# ADR-NNN: Title

**Status**: accepted

## Context

What was the situation that forced this decision? Include: technical constraints, scale,
team knowledge, timeline, alternatives that were available at the time.

## Decision

What was decided, in active voice. "We will use X." One clear statement, then supporting
detail if needed.

## Consequences

- **Easier**: what becomes simpler as a result
- **Harder**: what is constrained or made more complex
- **Technical debt**: any shortcuts taken that should be revisited
~~~

Valid status values: `proposed` | `accepted` | `deprecated` | `superseded by ADR-NNN`

## Detecting architectural decisions

Read these signals to find decisions worth recording:

| Signal | Decision to document |
|---|---|
| Database client import (`psycopg2`, `mongoose`, `sqlx`, `aiosqlite`) | Database choice and type (relational vs document vs in-memory) |
| Auth middleware (`jwt`, `passport`, `djangorestframework-simplejwt`) | Authentication approach |
| Web framework (`fastapi`, `express`, `gin`, `actix-web`) | Framework choice |
| Message queue client (`kafka`, `rabbitmq`, `celery`, `bullmq`) | Async / event-driven communication |
| ORM vs raw SQL (`sqlalchemy`, `prisma`, `typeorm` vs `db.execute(...)`) | Data access pattern |
| Monorepo tooling (`turbo`, `nx`, `lerna`) | Monorepo structure decision |
| `Dockerfile` / `docker-compose.yml` | Deployment model |
| `tests/unit/` + `tests/integration/` split | Test strategy |
| `services/`, `domain/`, `infra/` directory pattern | Layered architecture / DDD |
| Multiple stacks in subdirectories | Polyglot architecture split |

Focus on decisions with consequences: a choice that constrains future contributors or that a reasonable engineer might have made differently.

## Context section: what to write when the rationale is unclear

If you can observe the decision but not the reason (no comments, no commit messages, no docs):

~~~markdown
## Context

> [Unable to determine from code alone — fill in the business or technical constraint
> that drove this choice.]

The system uses X rather than Y. [Describe what you can observe: scale characteristics,
existing dependencies, team conventions.]
~~~

Do not fabricate context. A partially-filled ADR with an honest placeholder is more useful than a confidently-wrong explanation.

## When to ask before writing

Ask targeted questions only for **surprising** decisions — where a reasonable engineer would have defaulted to something different:

- Ask: chose Redis as the primary data store (most projects default to a relational DB)
- Ask: chose SQLite in a service that appears to need multi-instance scale
- Ask: two different ORMs used in the same codebase (unusual split — why?)
- Don't ask: chose PostgreSQL for relational data (the obvious default)
- Don't ask: chose React for the frontend (the dominant default)

Ask at most **5** questions across all ADRs in one session (see Step 3.5 exception in SKILL.md). Batch them in a single message. For the rest, use the placeholder pattern above.

## Handling existing ADRs

If `docs/adr/` already has files:

1. Read all of them before writing anything
2. Check each against the current code — does the decision still match?
   - Still accurate → leave it alone
   - Decision was reversed → create a new ADR that supersedes it; update the old one's status to `superseded by ADR-NNN`
   - Decision was never implemented → update status to `proposed` or `rejected`
3. Only write new ADRs for decisions not already documented

## How many ADRs to write

One ADR per meaningful architectural decision, not one per file or component. Aim for 3–8 for a typical project. More than 10 in a first pass usually means some should be merged or omitted.

**Worth an ADR:**
- Technology or library choices that are hard to reverse
- Structural patterns that constrain how the codebase grows
- Trade-offs that future contributors need to understand to work safely

**Not worth an ADR:**
- Naming conventions or code style choices
- Which specific version of a dependency to use
- Implementation details within a single module
