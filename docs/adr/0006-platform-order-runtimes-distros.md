# 0006. Platform order, Linux runtimes, and supported distros

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

Phase 1 builds complete tooling for one platform before the next. The order, the Linux runtimes, and the test distros must be fixed.

## Decision

Driver order: Docker (remote host over SSH) → Linux native (systemd) → Kubernetes (Helm SDK) → AWS (ECS Fargate first). Linux runtimes: generic binary → Java → Node.js, built on one runtime plugin pattern; Python and .NET later. Supported distros in Phase 1: Ubuntu 22.04/24.04 LTS, Rocky/Alma 9, Debian 12, Amazon Linux 2023.

## Consequences

- Docker proves the spec, driver interface, SSH transport, and state with the least platform work.
- Linux native reuses the SSH transport.
- SELinux and AppArmor are tested early.
- Each driver supports deploy, update, rollback, destroy, and status before the next starts.
