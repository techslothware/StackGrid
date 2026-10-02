# 0014. Monitoring split across phases

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

"Monitoring" covers deploy health checks, StackGrid self-observability, and ongoing client app monitoring.

## Decision

Deploy health checks (with automatic rollback) are built in Phase 1. Self-observability (structured logs, OpenTelemetry, Prometheus `/metrics`) is built in Phases 1–2. Client app monitoring (uptime, alerts, drift, version overview) is Phase 6, and integrates with existing tools rather than replacing them.

## Consequences

- Safe rollback is possible from the first release.
- Phase 6 can alert per client, product, and license tier.
