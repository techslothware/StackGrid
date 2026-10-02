# 0012. OIDC and API tokens, three-role RBAC

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

The API must be secure for human users and for machines (CI, webhooks, integrations).

## Decision

Humans authenticate with OIDC (Keycloak, Entra ID, Okta, Google). Machines use scoped API tokens: hashed at rest, expiring, revocable. StackGrid stores no passwords. RBAC roles `admin`, `operator`, `viewer`, bound at org → environment scope. Every mutating call writes an audit log entry. Later phases add permissions (for example `pii:read`).

## Consequences

- No password, MFA, or reset handling in StackGrid.
- Each install needs an OIDC provider.
