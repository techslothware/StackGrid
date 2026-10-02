# 0020. Client PII protection

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

Client PII must be kept separate from deployment state (GDPR, SOC 2).

## Decision

Store PII in separate tables; deployment data references clients by opaque ID only. Encrypt PII fields with keys from the secrets provider (Vault / AWS KMS). Erase by crypto-shredding. Add the `pii:read` permission. Support per-org retention and data export.

## Consequences

- Operators can deploy without seeing PII.
- Audit and deployment history survive erasure.
- Each self-hosted vendor is the data controller; StackGrid supplies the controls, not a certification.
