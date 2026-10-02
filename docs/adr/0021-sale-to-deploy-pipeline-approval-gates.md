# 0021. Sale-to-deploy pipeline with approval gates

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

Sales should lead to deployment with as little manual work as possible, but some deals need checks.

## Decision

Build an automated pipeline: prospect → quote → signed/paid → client created → license issued → deployed → health verified → status updated. Each product defines its own approval gates. E-signature uses integrations (DocuSign, Dropbox Sign), not a built-in tool.

## Consequences

- Small products can run fully automatic; large ones stay controlled.
- The workflow spans days, so Phase 5 reviews River vs Temporal.
