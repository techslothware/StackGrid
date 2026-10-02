# 0007. State store interface: SQLite for CLI, Postgres for API

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

Rollback and status need a record of what is deployed where, at which revision.

## Decision

Define a state store interface. Phase 1 ships a SQLite backend for the CLI. Phase 2 adds a Postgres backend for the API. Both use the same schema, with `org_id`.

## Consequences

- The CLI stays a single binary with no setup.
- Drivers do not change when the backend changes.
