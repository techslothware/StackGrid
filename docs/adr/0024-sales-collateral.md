# 0024. Sales collateral and product docs

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

The sales layer needs product documentation.

## Decision

Generate sales collateral (datasheets, feature lists, pricing sheets) from Product layer data. Link to the vendor's existing product docs through a docs URL on each release manifest. Do not host full product docs.

## Consequences

- Collateral stays in sync with the catalog and pricing.
