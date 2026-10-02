# 0022. StackGrid distribution

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

Vendors must install StackGrid itself.

## Decision

Ship three install methods, added in step with the drivers: single binary + Postgres as a systemd service (Phase 1–2), container image + Docker Compose (Phase 2), Helm chart (Phase 3). StackGrid deploys itself with its own drivers.

## Consequences

- Self-deployment is an end-to-end test of the drivers.
