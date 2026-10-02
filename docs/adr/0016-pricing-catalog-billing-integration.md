# 0016. Price catalog in StackGrid, billing by integration

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

Products need pricing. Billing, payments, and tax are a large, regulated domain.

## Decision

Phase 3 stores a price catalog (plans, tiers, add-ons, currencies, billing periods). Phase 5 integrates billing tools (Stripe first) through the provider pattern. StackGrid does not invoice, take payments, or calculate tax.

## Consequences

- Quotes use the catalog.
- Payment events from the billing tool can trigger the sale-to-deploy pipeline.
