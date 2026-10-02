# 0002. Push transport first, pull agent on demand

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

StackGrid must reach target environments (Linux hosts, Docker hosts, Kubernetes clusters, AWS accounts). It can connect out to them (push) or run an agent inside them that calls home (pull).

## Decision

Use push first: StackGrid holds credential references and connects out over SSH, the Kubernetes API, and the AWS API. Hide the transport behind an interface. Design the gRPC agent protocol in Phase 2, but build the pull agent only when a client blocks inbound access.

## Consequences

- Fast to build and test; matches how Helm, kubectl, and Terraform work.
- Clients that block inbound access are not supported until the agent exists.
- AWS access uses cross-account IAM role assumption, never stored access keys.
