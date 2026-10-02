# 0023. Phase exit criteria

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

The build is bottom up. Each phase must be complete before the next starts.

## Decision

Every phase ends with: features working end to end on its test matrix; unit and integration tests in CI on real targets; user guide, API/CLI reference, and ADRs; a security review; a tagged release (`v0.1` = Phase 1 … `v1.0` = Phase 5). Track each phase as a GitHub milestone with one issue per feature.

## Consequences

- Clear, testable phase boundaries.
- Decisions stay recorded in `docs/adr/`.
