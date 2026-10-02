# 0019. Client data ownership and CRM sync

- **Status:** Accepted
- **Date:** 2026-10-02

## Context

Many vendors already use a CRM (Salesforce, HubSpot). Others have none.

## Decision

StackGrid owns client records. Optional CRM sync connectors map CRM fields to StackGrid fields. Each field has one owner (CRM or StackGrid), so there are no two-way conflicts.

## Consequences

- Vendors without a CRM can use StackGrid directly.
- Connector work follows the provider pattern.
