# 0001. Self-hosted first, SaaS-ready

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

StackGrid could run as a self-hosted install per vendor, or as a multi-tenant SaaS that many vendors share. The choice affects credentials, tenancy, and security in every phase.

## Decision

Ship StackGrid as a self-hosted product first: one vendor per install. Every record carries an `org_id` from day one so a multi-tenant SaaS mode stays possible later.

## Consequences

- Phases 1–2 stay simple. StackGrid does not hold other companies' cloud credentials early.
- All schemas and queries must scope by `org_id`, even when only one org exists.
- SaaS mode is a backlog item and needs its own tenant isolation review.
