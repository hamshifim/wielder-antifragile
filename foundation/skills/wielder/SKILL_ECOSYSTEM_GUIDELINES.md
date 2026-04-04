---
description: Wielder Multi-Surface Ecosystem Architecture Guidelines
---

# Wielder Ecosystem Guidelines

The fundamental architectural principle behind the Wielder `ecosystem` framework is to actively facilitate **multi-cloud and multi-surface topologies**. Execution paths natively span across disparate infrastructural foundations without modifying the core business logic.

### 1. The Multi-Surface Abstraction
Wielder never forces operations to be localized exclusively to one cloud provider or physical machine natively. "Ecosystems" fundamentally decouple the source execution environment (e.g., `local_wsl`, `workstation_mac`) from the target operational endpoints (e.g., `Google Workspace`, `AWS S3`, `GCP Storage`). 
- **Guideline:** Pipelines and central provisioners MUST actively trace and report explicitly which surface is invoking the logic, and which remote surface is catching the payload. 
- **Guideline:** Logging must rigorously reflect structural cross-surface bounds natively: e.g. `[workstation_wsl => Google Workspace] Provisioning target...`.

### 2. Autonomous Ecosystem Provisioning (The `provision` block)
The `ecosystem` definitions explicitly maintain their own isolated network configurations (e.g., `ecosystem_manifest.conf`) which structurally dictate their target surfaces and foundational boundaries natively.
- **Guideline:** Target deployment keys, dependency paths, and foundational infrastructure variables must be universally aggregated natively inside the explicit `provision` dictionary exactly at the root of the targeted ecosystem configuration mappings. 

### 3. Decoupling Artifact Execution via Bucketeer Maps
The native `ecosystem_map` physically bridges topological environments across network boundaries dynamically eliminating source-code coupling structurally!
- **Guideline:** The system can evaluate local filesystem boundaries natively via `DevBucketeer` locally, and dynamically shift gears exclusively onto `GoogleBucketeer` mapping perfectly to explicit multi-cloud artifact deployments dynamically orchestrating the identical logic across boundaries seamlessly!

### 4. Ecosystem as the Home of Execution Topology
Ecosystems are not merely deployment labels. Their deeper role is to carry the **execution topology** of the workload: which dependencies are local, which are remote services, and which combinations form a hybrid.
- **Guideline:** If an application can satisfy a dependency through multiple modes such as local SDK execution, a direct API call to a colocated service, or a hybrid local/remote arrangement, that choice belongs in the ecosystem layer.
- **Guideline:** `stage_tier` is not the right home for these decisions. `stage_tier` describes deployment maturity (`dev`, `stage`, `prod`), whereas execution topology describes dependency distribution and service access shape.
- **Guideline:** Favor ecosystems to express combinations of surfaces. This includes pure local execution, local microservice execution, and future mixed topologies where one dependency remains local while another resolves against a remote surface.
- **Guideline:** A local-first base ecosystem is valid and often preferable during active development. Additional ecosystems may then layer service-oriented or hybrid variants without forcing application business logic to absorb deployment details.
- **Guideline:** Application code should read the already-resolved topology from the ecosystem configuration and branch at the transport boundary only. Scientific DAG payloads should not be overloaded with infrastructure routing concerns.
