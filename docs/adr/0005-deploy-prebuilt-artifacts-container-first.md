# 0005. Deploy pre-built artifacts, container-first

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

StackGrid could build software from source per app type, or only deploy artifacts the vendor already built.

## Decision

StackGrid deploys pre-built artifacts only. Container images are supported on every platform. Native artifacts (binary, JAR, Node.js tarball) are supported only by the Linux driver. Artifacts are pinned by digest or checksum.

## Consequences

- Building from source (CI) is out of scope; vendors keep their own CI.
- The app-type matrix only matters for bare Linux.
