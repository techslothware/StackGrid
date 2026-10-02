# 0008. Secrets provider interface with references

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

Deployments need target credentials (SSH keys, kubeconfig, IAM role ARNs) and app secrets.

## Decision

Specs reference secrets by name (`secret://<name>`), never inline values. A secrets provider interface resolves them at deploy time. Phase 1 backends: env vars and SOPS/age encrypted files. Phase 2 adds HashiCorp Vault and AWS Secrets Manager. Each platform injects secrets natively (Docker env/secrets, systemd `EnvironmentFile` mode `0600`, Kubernetes Secrets, ECS from Secrets Manager).

## Consequences

- Specs are safe to commit and to store in the database.
- The same provider pattern is reused for licensing, billing, and CRM.
