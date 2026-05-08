# Wieldable Functionalities

This catalog tracks reusable operator-facing capabilities that should be driven through Wielder configuration, entrypoints, and provider accessors rather than ad hoc shell memory.

## Catalog Rule

Add a functionality here when it becomes a repeated Starget operation with a stable command shape, config contract, or provider abstraction. Keep detailed mechanics in the owning skill, app README, or task markdown; this file is the index.

## Current Functionalities

### Terraform Provisioning

- Purpose: provision long-lived cloud assets and project resources.
- Shape: app-owned Wielder entrypoints wrap Terraform `plan`, `apply`, and `delete`.
- Contract: environment-specific values originate in HOCON and render into Terraform variables.
- Operator rule: run `plan` in-agent when lightweight; hand off long `apply` to the operator terminal unless explicitly asked to wait.

### Kubernetes Workloads

- Purpose: deploy, inspect, scale, and delete Wielder-managed Kubernetes workloads.
- Shape: workflow/app entrypoints own Kubernetes manifests, ConfigMaps, Jobs, Deployments, and local access bridges.
- Contract: runtime topology resolves through ecosystem, surface, stage tier, and app config rather than raw `kubectl` fragments.
- Operator rule: prefer Wielder entrypoints for managed resources; use raw `kubectl` mainly for evidence gathering and triage.

### Storage Cloning

- Purpose: mirror or migrate data between local paths, Google Drive, Google Cloud Storage, and AWS S3.
- Shape: `WCloner` receives typed endpoints from HOCON and delegates destination creation to Bucketeer factories.
- Contract: source, sink, sync type, WClone backend config path, flags, RBAC, and create-if-missing behavior live in config.
- Runtime security: hosted daemons should read external credentials from provider-native secret managers through a dedicated runtime identity, with bucket and pub/sub access scoped to the exact resources they operate on.
- Operator rule: agents may run `show`, `plan`, and `probe`; long `apply` syncs should be handed to the operator terminal.

### WClone Backend Configuration

- Purpose: avoid repeated interactive setup for stable clone backends.
- Shape: Wielder scripts may call `WCloner.configure_wclone(...)` to ensure generic local remotes such as `starget_aws` and `starget_gcs` in a developer-local backend config file.
- Contract: generated configs live under `~/.config/wclone/`; credentials should come from runtime auth, MFA helpers, service account flows, or short-lived environment variables.
- Operator rule: do not ask the operator to manually recreate deterministic clone remotes when the script can create the safe non-secret sections.
- GCP rule: local dev may inject `gcloud auth print-access-token`; GCP-hosted jobs should use service-account ADC or metadata credentials with backend `env_auth = true`.
- Secret rule: version non-secret backend shape only. Secret containers and IAM are provisionable; secret payloads are operator/runtime state.

### WArgus Secret Guarding

- Purpose: pump local/operator secret values into provider-native secret managers and expose security operations through a Wielder surface.
- Shape: `WArgus` is a provider-neutral factory/interface with implementations such as `GCPArgus`, `GoogleWorkspaceArgus`, `AWSArgus`, and local/dev variants.
- Contract: secret definitions, target provider, stage tier, Workspace group ownership, runtime readers, and payload sources live in config; payload values are redacted from logs.
- Security hood: default hood is `org`; non-default hoods such as `restricted` or `break_glass` become explicit identity name fragments when they change access.
- Boundary: WArgus owns reusable secret operations; Terraform/provisioning owns durable secret containers, IAM bindings, service accounts, and provider resources.
- Workspace boundary: `GoogleWorkspaceArgus` owns group-oriented operations such as verifying/provisioning Workspace groups and group-bound access intent; `GCPArgus` owns GCP Secret Manager payload operations.
- Operator rule: `show` and `plan` must never print payloads; `apply` may populate secret versions from approved local secret sources.

## Candidate Functionalities

- Cloud pub/sub provisioning and event contract wiring.
- Scheduled jobs and cron-like cloud executions.
- Static resource bucket creation and replication.
- Workflow image build and publication.
- Cross-cloud bucket migration handoffs.
