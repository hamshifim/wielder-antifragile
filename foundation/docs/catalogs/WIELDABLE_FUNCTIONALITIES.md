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
- Contract: source, sink, sync type, rclone config path, flags, RBAC, and create-if-missing behavior live in config.
- Operator rule: agents may run `show`, `plan`, and `probe`; long `apply` syncs should be handed to the operator terminal.

### Rclone Configuration

- Purpose: avoid repeated interactive rclone setup for stable remotes.
- Shape: Wielder scripts may call `WCloner.configure_rclone(...)` to ensure generic local rclone remotes such as `starget_aws` and `starget_gcs` in a developer-local config file.
- Contract: generated configs live under `~/.config/rclone/`; credentials should come from runtime auth, MFA helpers, service account flows, or short-lived environment variables.
- Operator rule: do not ask the operator to manually recreate deterministic rclone remotes when the script can create the safe non-secret sections.
- GCP rule: local dev may inject `gcloud auth print-access-token`; GCP-hosted jobs should use service-account ADC or metadata credentials with rclone `env_auth = true`.

## Candidate Functionalities

- Cloud pub/sub provisioning and event contract wiring.
- Scheduled jobs and cron-like cloud executions.
- Static resource bucket creation and replication.
- Workflow image build and publication.
- Cross-cloud bucket migration handoffs.
