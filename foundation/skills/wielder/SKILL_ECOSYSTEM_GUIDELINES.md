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

### 5. Workflow Contracts and Ecosystem Ownership
Messaging topics, consumer groups, and similar workflow orchestration contracts should not default to project-global scope.
- **Guideline:** If a workflow currently maps 1:1 to an ecosystem, place its orchestration contract inside that ecosystem.
- **Guideline:** If the same workflow contract must be shared by multiple ecosystems later, promote it into a workflow-level include and let the participating ecosystems inherit it.
- **Guideline:** Do not keep workflow-specific messaging contracts at project root unless they are truly global across the entire project.

### 6. Naming App-Specific Ecosystem Overrides
Cross-project orchestration may need to tell multiple downstream apps which ecosystem they should run under. A generic field name can become ambiguous once one config modulates more than one app.
- **Guideline:** Strongly suggest using app-specific override names such as `app_ecosystem_mmseqs_sequence_alignment` when a config may route multiple downstream apps.
- **Guideline:** Strongly suggest avoiding generic names such as `app_ecosystem` in multi-app orchestration layers, because they tend to hide which downstream runtime is being modulated.
- **Guideline:** Strongly suggest avoiding self-referential assignments where an ecosystem overlay redundantly sets an app-specific ecosystem field to the exact same ecosystem name as the containing overlay, unless that duplication is deliberately carrying review signal.

### 7. Family Ecosystems vs Bootable Runtime Ecosystems
Thin reusable ecosystem families and concrete bootable ecosystems are related, but they are not the same thing.
- **Guideline:** Strongly suggest treating a shared family ecosystem such as `protenix_binding` as a reusable semantic base first, not as an automatic downstream runtime target.
- **Guideline:** Strongly suggest using `app_ecosystem_<app_name>` as the explicit bridge when a concrete deploy ecosystem needs to adapt a downstream app onto a different bootable runtime ecosystem.
- **Guideline:** Strongly suggest avoiding the temptation to point a downstream runtime directly at an abstract family ecosystem unless that family has intentionally been made bootable and verified end to end.
- **Guideline:** When a family ecosystem exists alongside thin concrete ecosystems, strongly suggest keeping the concrete ecosystems responsible for operational facts such as registry authority, kube context, pull behavior, and downstream runtime bridging.

### 8. Thin Concrete Ecosystems
Once a shared family ecosystem has been extracted, the remaining concrete ecosystems should stay obvious and reviewable.
- **Guideline:** Strongly suggest consolidating concrete override ecosystems into a single `ecosystem_manifest.conf` when their remaining role is to override a small set of operational facts.
- **Guideline:** Strongly suggest deleting empty or misleading concrete override fragments once their contents have been centralized, rather than preserving them as ceremonial files.
- **Guideline:** Strongly suggest keeping deployment-resource ordering and Kubernetes object lists at the deployment layer, not inside ecosystem overlays, even when those ecosystems become very thin.
