# 0017. Release manifest links products to deployments

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

The Product layer must turn a product version into deployments on any supported platform.

## Decision

Each product version has an immutable release manifest: pinned artifacts, a spec template per platform, a config schema (JSON Schema) for per-client values, feature mapping, upgrade rules, an optional tenant hook, and a docs URL. Product version + target + client values render a Phase 1 spec, which goes to the Phase 2 API.

## Consequences

- Phases 1–2 only ever see rendered specs and need no change.
- Fixes require a new version.
