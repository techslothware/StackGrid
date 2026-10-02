# 0009. Single-tenant deployments first, tenant hook later

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

A vendor's software may run as one instance per client (single-tenant) or as one shared app (multi-tenant).

## Decision

Phase 1 drivers deploy dedicated instances per client. Isolation level (shared cluster with namespaces vs dedicated cluster or account) is a per-target choice. Multi-tenant products are supported later through a tenant provisioning hook (webhook or script) in the release manifest (Phase 3).

## Consequences

- Drivers stay focused on what they do natively.
- Multi-tenant SaaS vendors stay in scope without changes to Phase 1.
