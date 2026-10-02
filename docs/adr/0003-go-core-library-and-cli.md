# 0003. Go core library and CLI with a declarative spec

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

Phase 2 must call the Phase 1 tooling. The form of that tooling sets the contract between the phases. A language is also needed for the core.

## Decision

Write the core in Go as a library (`pkg/`) with a driver interface, plus a CLI (`stackgrid`) that uses it. Every deployment is described by a declarative, versioned spec. The Phase 2 API imports the same library and uses the spec as its request body. Use a Go monorepo (`cmd/`, `internal/`, `pkg/`, `web/`, `deploy/`, `docs/`).

## Consequences

- Each platform can be tested from the CLI before any API exists.
- Single static binary; the same language suits a future agent.
- Go has native SDKs for Docker, Kubernetes, Helm, AWS, and Terraform execution.
