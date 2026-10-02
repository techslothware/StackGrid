# 0018. Vendor staff only access

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

The original overview included a client onboarding portal.

## Decision

Only the vendor's staff use StackGrid. The vendor's end clients never log in. The onboarding portal is removed from scope.

## Consequences

- No untrusted user type; simpler auth and tenant isolation.
- Client data is entered by staff or synced from a CRM.
