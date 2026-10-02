# 0004. Native Go SDKs; Terraform for AWS infrastructure only

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

Drivers can wrap external tools (Ansible, Terraform, Helm CLI) as subprocesses or use Go SDKs directly.

## Decision

Use native Go SDKs: `golang.org/x/crypto/ssh` for Linux, the Docker SDK, `client-go` and the Helm SDK for Kubernetes, and the AWS SDK v2. Use Terraform (via `terraform-exec`) only to provision AWS infrastructure. Do not use Ansible.

## Consequences

- The binary stays self-contained, with typed errors for the API.
- Terraform adds a runtime dependency, but only for the AWS driver, where its state and drift handling are worth it.
- Terraform state lives in S3 with a DynamoDB lock, separate from StackGrid state.
