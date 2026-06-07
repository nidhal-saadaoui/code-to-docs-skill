# ADR-001: SQLite over PostgreSQL

**Status**: accepted

## Context

Beacon's write pattern is append-only and low-frequency: one row per registered service per polling interval (default 60 seconds). Even with 100 registered services, that is under 2 writes per second at steady state. Reads are similarly light — health history queries return at most a few hundred rows.

The service is deployed as a single instance inside a private network. There is no requirement for horizontal scaling, replication, or concurrent writes from multiple processes.

## Decision

We will use SQLite via `aiosqlite` as the sole data store.

## Consequences

- **Easier**: zero operational overhead — no database server to provision, no connection string secrets to manage, no migration tooling needed for the schema. The entire state is a single file that can be backed up with `cp`.
- **Harder**: if beacon ever needs to run as multiple replicas behind a load balancer, SQLite's single-writer model becomes a hard constraint. A migration to PostgreSQL would require rewriting the data layer.
- **Technical debt**: hand-written SQL instead of an ORM means schema changes require careful manual migration. Acceptable at current scale; revisit if the schema grows beyond 5 tables.
