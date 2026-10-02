# 0015. Licensing provider interface

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

Products need licensing. Some vendors already have a license tool; others have none.

## Decision

Define a licensing provider interface. The built-in provider issues signed offline license keys (Ed25519) first, then adds an online license server (activation, usage reporting, revocation). Licenses are injected into deployments as secrets. External providers integrate a vendor's existing license tool.

## Consequences

- Works for air-gapped and single-tenant installs.
- **Open item:** the external provider contract (for example Keygen, Cryptlex, LicenseSpring, custom) is defined later.
