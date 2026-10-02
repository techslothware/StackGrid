# 0010. Job execution with River on Postgres

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

Deployments take minutes and need retries, timeouts, and rollback. Options were Temporal, a Postgres-backed queue, or a message broker with custom workers.

## Decision

Use River (Postgres-backed queue for Go) in Phase 2. Model each deployment as an explicit step state machine stored in Postgres. Keep a job interface so Temporal can replace River later.

## Consequences

- Install stays one binary + Postgres.
- Retry and rollback steps are our own code.
- Review in Phase 5, when the sale-to-deploy workflow spans days.
