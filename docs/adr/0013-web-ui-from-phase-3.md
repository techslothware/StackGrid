# 0013. Web UI from Phase 3 onward

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

Operators can use a CLI, but product, client, and sales users need screens.

## Decision

Phases 1–2 are CLI + API only. The web UI starts in Phase 3 and grows with each phase. Stack: TypeScript + React, API client generated from OpenAPI, served as static files by the Go binary.

## Consequences

- Each phase from 3 ships API + UI together.
- The one-binary install is kept.
