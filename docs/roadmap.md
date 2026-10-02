# StackGrid Roadmap

StackGrid lets a software vendor manage its products from sale to deployment in one place:
**Sales → Client → Product → Deployment API → Deployment tooling**, plus monitoring.

The build order is **bottom up**. Each phase is complete, tested, and released before the next
phase starts. Later phases build on the stable interfaces of earlier phases.

Decisions behind this roadmap are recorded as ADRs in [`docs/adr/`](adr/README.md).

---

## 1. Guiding principles

| Principle | Meaning | ADR |
| :-- | :-- | :-- |
| Self-hosted first, SaaS-ready | Each vendor runs its own StackGrid. Every record has an `org_id` so a multi-tenant SaaS mode stays possible. | [0001](adr/0001-self-hosted-first-saas-ready.md) |
| Small install | One Go binary + Postgres. No extra mandatory services. | [0003](adr/0003-go-core-library-and-cli.md), [0010](adr/0010-job-execution-river-on-postgres.md) |
| Declarative spec | Every deployment is described by one versioned spec. The CLI, API, and higher layers all produce or consume it. | [0003](adr/0003-go-core-library-and-cli.md) |
| Provider pattern | External concerns (secrets, licensing, billing, CRM, e-signature, monitoring) sit behind interfaces. A built-in provider exists where useful; external tools plug in. | [0008](adr/0008-secrets-provider-interface.md), [0015](adr/0015-licensing-provider.md) |
| Deploy, don't build | StackGrid deploys pre-built artifacts. The vendor's CI builds them. | [0005](adr/0005-deploy-prebuilt-artifacts-container-first.md) |
| Vendor staff only | End clients of the vendor never log in to StackGrid. | [0018](adr/0018-vendor-staff-only-access.md) |

### Out of scope

- Building software from source (CI).
- A client-facing onboarding portal.
- Built-in invoicing, payments, tax, or e-signature (integrations only).
- Hosting full product documentation (link to vendor docs only).

---

## 2. Phase overview

| Phase | Name | Release | Depends on |
| :-- | :-- | :-- | :-- |
| 1 | Deployment tooling (library + CLI) | `v0.1` | — |
| 2 | Deployment API | `v0.2` | 1 |
| 3 | Product layer (+ web UI starts) | `v0.3` | 2 |
| 4 | Client layer | `v0.4` | 2, 3 |
| 5 | Sales layer | `v1.0` | 3, 4 |
| 6 | Monitoring layer | `v1.1` | 2, 3, 4 |
| — | Backlog (pull agent, SaaS mode, more runtimes) | on demand | varies |

Minor milestones inside a phase use pre-release tags (for example `v0.1.0-docker`).

---

## 3. Phase 1 — Deployment tooling

**Goal:** a Go library and CLI that can deploy, update, roll back, tear down, and report status of
an application on Docker, Linux, Kubernetes, and AWS, from one declarative spec.

### 1.0 Core foundation

Build before any driver.

- **Monorepo layout:** `cmd/stackgrid` (CLI), `internal/` (private code), `pkg/` (public library API),
  `web/` (added in Phase 3), `deploy/` (install files), `docs/`.
- **Deployment spec** (`apiVersion: stackgrid.io/v1alpha1`):
  - `app` — name, version, artifact (image digest or native artifact + checksum), runtime.
  - `target` — platform, connection reference, placement (host, namespace, cluster, region).
  - `config` — env vars, ports, resources, volumes.
  - `secrets` — references only, `secret://<name>`. Never inline values.
  - `healthCheck` — HTTP/TCP/command check, timeout, success threshold.
  - JSON Schema for validation. The same schema is the Phase 2 API request body.
- **Driver interface** (`pkg/driver`): `Validate`, `Plan`, `Deploy`, `Status`, `Rollback`, `Destroy`.
  Drivers return typed errors and structured events (step, progress, log line).
- **Transport interface:** SSH, Kubernetes API, AWS API. Kept separate from drivers so a pull agent
  can replace push later ([0002](adr/0002-push-transport-pull-agent-on-demand.md)).
- **State store interface** (`pkg/state`): deployments, revisions, events, all with `org_id`.
  SQLite backend for the CLI ([0007](adr/0007-state-store-interface.md)).
- **Secrets provider interface** (`pkg/secrets`): env var and SOPS/age encrypted file backends
  ([0008](adr/0008-secrets-provider-interface.md)).
- **Health checks** after every deploy. Failed check → automatic rollback to the previous revision.
- **Observability:** structured logs (`slog`), OpenTelemetry traces and metrics from the start.
- **CLI:** `stackgrid validate | plan | deploy | status | rollback | destroy | history`.
- **CI:** lint, unit tests, build, release binaries (GoReleaser).

### 1.1 Docker driver (`v0.1.0-docker`)

- Remote Docker host over SSH (Docker Go SDK through an SSH tunnel).
- Pull image by digest, create/replace container, networks, volumes, restart policy.
- Secrets injected as env vars or Docker secrets.
- Rollback = restart previous image digest and config revision.
- Proves the spec, driver interface, SSH transport, state, and health checks end to end.

### 1.2 Linux native driver (`v0.1.0-linux`)

- Reuses the SSH transport from 1.1. Native Go SSH (`golang.org/x/crypto/ssh`), no Ansible.
- **Runtime plugin pattern:** install runtime → place artifact → write systemd unit → health check.
- Runtimes, in order:
  1. Generic binary (Go, Rust, any static executable).
  2. Java (fat JAR, JDK version pinned in spec).
  3. Node.js (tarball with `node_modules`, Node version pinned).
- Releases kept side by side (`/opt/<app>/releases/<version>`) with a `current` symlink for fast rollback.
- Secrets in a systemd `EnvironmentFile` with mode `0600`.
- Handle SELinux (RHEL family) and AppArmor (Ubuntu/Debian) correctly.

### 1.3 Kubernetes driver (`v0.1.0-k8s`)

- `client-go` + Helm SDK. A generic StackGrid Helm chart for spec-based apps; vendor charts allowed.
- Namespace per deployment (single-tenant default), Kubernetes Secrets, rollout status as health signal.
- Rollback through Helm revision history.

### 1.4 AWS driver (`v0.1.0-aws`)

- ECS Fargate first.
- Terraform (via `terraform-exec`) for infrastructure only: VPC, ECS cluster/service, load balancer,
  RDS if needed. Terraform state in S3 with a DynamoDB lock, separate from StackGrid state.
- AWS access only by cross-account IAM role assumption (`sts:AssumeRole` + external ID).
  No stored access keys.
- Secrets from AWS Secrets Manager injected by ECS.

### Supported matrix (Phase 1)

| Platform | Artifact types | Test environments |
| :-- | :-- | :-- |
| Docker | Container image | Docker on each supported distro |
| Linux native | Generic binary, Java JAR, Node.js tarball | Ubuntu 22.04/24.04 LTS, Rocky/Alma 9, Debian 12, Amazon Linux 2023 |
| Kubernetes | Container image (Helm) | kind, k3s in CI; EKS before sign-off |
| AWS | Container image (ECS Fargate) | Dedicated AWS sandbox account; LocalStack for unit-level checks only |

Linux test hosts: VMs (Vagrant or Multipass) for full tests; systemd-enabled containers for fast CI.

### Phase 1 exit criteria

- All five operations (deploy, update, rollback, destroy, status) pass on every row of the matrix.
- Failed health check triggers automatic rollback on every platform.
- Plus the [shared exit criteria](#9-definition-of-done-for-every-phase).

---

## 4. Phase 2 — Deployment API

**Goal:** a secure, multi-user HTTP API on top of the Phase 1 library. No driver changes.

### Features

- **REST + JSON**, defined OpenAPI-first. Go server stubs generated with `oapi-codegen`
  ([0011](adr/0011-rest-external-grpc-internal.md)).
  - `POST /api/v1/deployments` → `202 Accepted` + job ID.
  - `GET /api/v1/deployments/{id}`, `PUT …/{id}`, `DELETE …/{id}`, `POST …/{id}/rollback`.
  - `GET /api/v1/jobs/{id}` (status), `GET …/{id}/events` (SSE log stream).
  - Targets, credentials references, environments, API tokens, audit log endpoints.
  - Outgoing webhooks for deployment events (signed with HMAC).
- **Job execution:** River (Postgres-backed queue). Each deployment is an explicit step state machine
  stored in Postgres, with retries, timeouts, and rollback on failure
  ([0010](adr/0010-job-execution-river-on-postgres.md)).
- **Postgres state backend** with migrations. Same schema as the SQLite backend.
- **Authentication:** OIDC for humans (Keycloak, Entra ID, Okta, Google). Scoped API tokens for
  machines: hashed at rest, expiring, revocable ([0012](adr/0012-authn-oidc-and-api-tokens-rbac.md)).
- **Authorization (RBAC):** roles `admin`, `operator`, `viewer`, bound at org → environment scope.
- **Audit log:** every mutating call records who, what, when, and from where.
- **Secrets backends:** add HashiCorp Vault and AWS Secrets Manager.
- **CLI remote mode:** the CLI can target the API instead of running drivers locally.
- **Self-observability:** `/metrics` (Prometheus), OTel traces across API → job → driver, `/healthz`.
- **Install:** container image + Docker Compose (StackGrid + Postgres)
  ([0022](adr/0022-stackgrid-distribution.md)).
- **Agent prep:** define the gRPC agent protocol and confirm the transport interface supports it.
  Do not build the agent ([0002](adr/0002-push-transport-pull-agent-on-demand.md)).

### Security work

- TLS everywhere, secure defaults, rate limits, request size limits, input validation against the
  spec JSON Schema.
- Threat model of the API and job runner. Dependency and container image scanning in CI.
- Credentials for targets are only references; the job runner resolves them at run time.

### Phase 2 exit criteria

- The full Phase 1 matrix passes through the API, not only the CLI.
- RBAC and token scope tests for every endpoint.
- Plus the [shared exit criteria](#9-definition-of-done-for-every-phase).

---

## 5. Phase 3 — Product layer

**Goal:** model the vendor's products, releases, licensing, and pricing. Start the web UI.

### Features

- **Products:** name, description, type (single-tenant, multi-tenant, both), supported platforms.
- **Editions / tiers** and **features** (feature flags, usage limits, quotas).
- **Release manifest** per product version ([0017](adr/0017-release-manifest.md)):
  - Artifacts pinned by digest/checksum.
  - Spec template per supported platform.
  - Config schema (JSON Schema) for per-client values.
  - Feature mapping: licensed feature → config flag or env var.
  - Upgrade rules: allowed paths, migration hooks, min/max versions.
  - Optional tenant provisioning hook for multi-tenant products
    ([0009](adr/0009-single-tenant-first-tenant-hook.md)).
  - Docs URL (link to vendor docs).
  - Releases are immutable once published. Fixes create a new version.
- **Render flow:** product version + target + client values → Phase 1 spec → Phase 2 API.
- **Licensing provider interface** ([0015](adr/0015-licensing-provider.md)):
  - Built-in provider: signed offline license keys (Ed25519). Vendor apps verify with a public key.
  - Built-in online license server (activation, usage reporting, revocation) — later milestone in Phase 3.
  - Licenses injected into deployments as secrets.
  - **Open item:** integration with a vendor's existing license tool (e.g. Keygen, Cryptlex,
    LicenseSpring, custom). Define the external provider contract later.
- **Price catalog:** plans, tiers, add-ons, currencies, billing periods. No invoicing
  ([0016](adr/0016-pricing-catalog-billing-integration.md)).
- **Web UI starts** ([0013](adr/0013-web-ui-from-phase-3.md)): TypeScript + React, API client generated
  from OpenAPI, served as static files by the Go binary. Phase 3 screens: products, releases,
  licensing, pricing, and a read-only view of deployments.
- **Install:** Helm chart for StackGrid itself. StackGrid deploys itself with its own K8s driver.

### Phase 3 exit criteria

- Deploy a product release to each platform from the UI and API, using only the release manifest.
- Issue, verify, and revoke a license end to end.
- Plus the [shared exit criteria](#9-definition-of-done-for-every-phase).

---

## 6. Phase 4 — Client layer

**Goal:** the vendor's client records, linked to products, licenses, and deployments.

### Features

- **Clients:** organisation details, contacts, addresses, status (prospect, active, suspended, churned).
- **Entitlements:** which products, editions, features, and licenses each client has.
- **Client environments:** each client's deployment targets (their host, cluster, AWS account role).
- **Links:** client → entitlements (Phase 3) → deployments (Phase 2).
- **Data ownership:** StackGrid owns client records. Optional CRM sync connectors (Salesforce,
  HubSpot) with per-field ownership. No two-way conflicts
  ([0019](adr/0019-client-data-ownership-crm-sync.md)).
- **PII protection** ([0020](adr/0020-pii-protection.md)):
  - PII in separate tables. Deployment data references clients by opaque ID only.
  - Field-level encryption, keys from the secrets provider (Vault / AWS KMS).
  - Erasure by crypto-shredding (delete the client key).
  - Permission `pii:read`. Operators can deploy without seeing PII.
  - Per-org retention policy and data export.
- **Access:** vendor staff only ([0018](adr/0018-vendor-staff-only-access.md)).
- **UI:** client list, client detail (entitlements, licenses, deployments), onboarding actions.

### Phase 4 exit criteria

- Create a client, grant an entitlement, issue a license, and deploy, all from the UI.
- Crypto-shred a client and confirm PII is unreadable while audit history stays intact.
- Plus the [shared exit criteria](#9-definition-of-done-for-every-phase).

---

## 7. Phase 5 — Sales layer

**Goal:** sales tools and an automated sale-to-deployment pipeline.

### Features

- **Prospects / leads:** store potential clients and qualification data (compatibility, requirements).
- **Quotes:** built from the price catalog. Versioned. Export to PDF.
- **Sales collateral:** datasheets, feature lists, and pricing sheets generated from Phase 3 data,
  plus links to vendor docs per release ([0024](adr/0024-sales-collateral.md)).
- **Integrations** (provider pattern):
  - Billing: Stripe first (others later). Payment events trigger the pipeline.
  - E-signature: DocuSign, Dropbox Sign. Signed contract events trigger the pipeline.
  - CRM: extend Phase 4 connectors to deals and opportunities.
- **Sale-to-deploy pipeline** ([0021](adr/0021-sale-to-deploy-pipeline-approval-gates.md)):
  prospect → quote → signed/paid → client created → license issued → deployed → health verified →
  status updated. Each product defines its approval gates (for example auto-deploy trial tier,
  operator approval for enterprise).
- **Workflow engine review:** the pipeline spans days. Decide whether River is still enough or
  whether to move to Temporal behind the same job interface.

### Phase 5 exit criteria

- A signed or paid deal deploys automatically for a product with no gates.
- A gated product waits for approval and then deploys.
- Plus the [shared exit criteria](#9-definition-of-done-for-every-phase).

---

## 8. Phase 6 — Monitoring layer

**Goal:** ongoing monitoring of deployed client applications ([0014](adr/0014-monitoring-split.md)).

Already built earlier: deploy health checks (Phase 1) and StackGrid self-observability (Phases 1–2).

### Features

- Uptime and health checks per deployment, on a schedule.
- Alerts per client, per product, per license tier (email, Slack, webhook, PagerDuty).
- Drift detection: compare the running state with the recorded spec revision.
- Version overview: which clients run which release; upgrade candidates.
- Integrate with existing tools (Prometheus, Grafana, Datadog, OpenTelemetry). Do not replace them.

---

## 9. Definition of done (for every phase)

From [0023](adr/0023-phase-exit-criteria.md):

- All planned features work end to end on the phase's test matrix.
- Unit and integration tests run in CI. Integration tests use real targets (VMs, kind/k3s, AWS sandbox).
- Docs: user guide, API/CLI reference, and an ADR for each major decision.
- Security review of the phase (secrets, auth, input validation).
- Tagged release.
- Tracking: one GitHub milestone per phase, one issue per feature.

---

## 10. Backlog (on demand)

| Item | Trigger |
| :-- | :-- |
| Pull agent (gRPC) | A client blocks inbound access to its environment. |
| SaaS / multi-tenant StackGrid | Demand for a hosted offering. Needs tenant isolation review. |
| Python and .NET runtimes (Linux driver) | A vendor needs them. |
| More AWS targets (EKS-managed provisioning, EC2) | A vendor needs them. |
| Other clouds (Azure, GCP) | A vendor needs them. |
| External licensing providers | Open item from Phase 3. |
| More billing / CRM / e-signature providers | A vendor needs them. |

---

## 11. Open items

- External licensing provider contract (Phase 3).
- Workflow engine review: River vs Temporal (Phase 5).
- Exact spec schema fields — finalise in Phase 1.0 with an ADR.
