---
description: Wielder PyHocon Configuration Guidelines (Strict Architectural SOP)
---

# Wielder PyHocon Configuration Guidelines

The Wielder framework relies on a deterministic and structurally sound configuration layer. As data domains and topologies scale, adherence to these engineering constraints improves maintainability, preserves clear failure boundaries, and supports consistent structural abstraction.

Wielder configuration names enduring managed units as `apps`. A deployment is an operational expression of an app, and a workflow is the orchestrated harness that coordinates many apps, dependencies, and observers. The configuration tree should preserve that distinction clearly.

---

## Part 1: The Resolving Configuration Ecology

To support a distributed super-repo architecture where applications concurrently bridge different sets of bare-metal development workstations, remote K8s clusters, and localized shadow deployments, Wielder enforces a hyper-granular, predictable resolution hierarchy.

### The Resolving Configuration Hierarchy

The evaluation hierarchy relies on object-tree merging priorities. Overrides elegantly layer across domains, establishing local primacy sequentially against standard defaults. The structure parsed functionally via `wield_conf.py` resolves chronologically from bottom (lowest fallback) to top (absolute primacy):

1. **Application Baseline (`app.conf`)**
   * **Role**: Defines the fundamental configuration limits for an application.

2. **Project Base (`project.conf`)**
   * **Role**: Root topology configuration spanning structural dependencies globally without overriding local `stage_tier` limits.

3. **Ecosystem & Deployment Triggers (`canary`, `destroy`, `ecosystem`)**
   * **Role**: Dictates global network routing, architecture maps, and overarching deployment footprint triggers spanning multiple domains.

4. **Domain Modes (`surface`, `stage_tier`, `security`)**
   * **Role**: The core operational matrix orchestrating exact multidimensional overrides. Defines physical orchestration parameters (`surface`), deployment boundaries (`dev`, `stage`, `prod`), and RBAC compartmentation.

5. **Context Packs (`context_conf/<name>/developer.conf`)**
   * **Role**: Centralized developer context packs dictating localized overrides during active development. These packs trump downstream domain boundaries without fragmenting overrides across repo-local `conf/developer/` folders.

6. **CLI Parser (Absolute Primacy)**
   * **Role**: Natively dictates execution execution, overriding developer configurations implicitly during dynamic pipeline spins without manual parameter passing inside sub-scripts.

### Architectural Mandates
* **No Orphaned Variables**: A configuration typically needs to exist natively inside the Application Baseline (Tier 1) before it can be overridden in Tier 4.
* **Fail-Closed Execution**: If an override requires structural evaluation arrays (`[]`), the Baseline establishes the empty array. Masking empty configurations inside comments to "bypass PyHocon overlays" is an antipattern.
* **Mock Isolation**: Experimental boundaries ride explicitly on the Domain Modes (`stage_tier`), avoiding pollution of Tier 1.

---

## Part 2: Configuration Guidelines (SOPs)

### 1. Strict Attribute Resolution
Configuration dependencies should guide the execution toward immediate awareness if environment parameters are missing.
- **Guideline:** Avoid wrapping configuration extraction blocks in generic `try...except Exception` silencing patterns. 
- **Guideline:** Prefer extracting properties natively via dot-notation (e.g., `conf.ecosystem`). Allow PyHocon to natively trigger a `ConfigMissingException` or `AttributeError` trace upon failure. Suppressing missing configurations obscures orchestration defects.

### 2. The Evaluation Trump Model (Ingrained CLI Architecture)
Historically, the Wielder architecture minimized CLI bindings to protect the mathematical purity of the `.conf` file. However, structurally isolating the CLI from the PyHocon loader creates catastrophic Execution Fragmentation (the "Splintering Source of Truth") where scripts evaluate configurations differently depending on how they were executed.
- **Guideline:** The centralized Antifragile parser (`get_ecosystem_parser()`) MUST be natively ingrained into the absolute bottom of the `get_starget_conf()` evaluation loop. 
- **Guideline (The Trump Card):** The evaluation hierarchy is mathematically absolute: *Project Base -> Ecosystem -> App -> Developer -> CLI*. By executing the CLI parser internally, terminal arguments natively map onto the configuration ConfigTree, systematically trumping all local developer configurations uniformly across every single orchestrating script without requiring manual `argparse` implementations in leaf files.
- **Cross-Reference:** This native PyHocon trump execution is the foundational bridge permitting safe dry-run Sandboxing natively. See the strict Staging Sandbox bounds mapped formally in [Wielder Imager & Staging Sandboxing](file:///home/gideon/starget/wielder-antifragile/foundation/skills/wielder/SKILL_WIELDER_IMAGER.md).
- **Guideline (Workflow Runtime Preference):** For distributed workflow ecosystems, CLI trumps are useful during bootstrap and planning, but runtime components should prefer staged configuration artifacts when native propagation exists. Avoid making container command-line flags the long-term operational source of truth when `context_conf/<name>/developer.conf` or another staged config artifact can be copied into the runtime surface.
- **Guideline (Accessor Preference):** Strongly suggest resolving application or service configuration through the canonical accessor (`get_app_conf()`, `get_service_conf()`, or peer helpers) rather than manually reconstructing HOCON layers inside leaf scripts.
- **Guideline (One Accessor Doctrine):** There should be one canonical accessor path for a given config contract. This is not aesthetic minimalism. It preserves one source of truth, one resolution order, one predictable failure boundary for missing variables, and one place to debug precedence. Multiple accessors for the same contract create hidden override paths, unreadable local-vs-ecosystem-vs-app precedence, and eventual configuration drift across modules.
- **Guideline (Operational Reason):** The single accessor rule exists to preserve reproducibility, debuggability, and locality of reasoning in a multi-repo, multi-agent system. If two helpers can resolve the same contract from different file sets, debugging becomes forensics instead of engineering.
- **Guideline (Cross-Repo Ownership):** In a super-repo, the orchestrating repo may need to read config owned by a child repo. That is valid, but ownership must stay explicit. Load the child repo's app conf through the child repo's own canonical accessor, then extract only the owned subtree or fields required by the orchestrator.
- **Guideline (No Mega-Conf Mixing):** Do not merge a whole foreign app tree into the local execution conf just to obtain a few image paths, ports, or service fields. Read the foreign owned contract, extract the minimal fields needed, and keep execution identity in the local caller conf.
- **Guideline (False Genericity):** Strongly suggest resisting the urge to create "generic" cross-repo config loaders inside `Wielder`. Many cross-repo relationships are domain-specific and idiosyncratic. If repetition emerges, abstract it locally in the orchestrating repo, not in the generic framework.
- **Guideline (Leaf Ownership Bridges):** When a leaf module already knows exactly which foreign app it wields, it should depend directly on that foreign app's canonical accessor rather than routing the dependency through a broader orchestrator helper. This keeps ownership honest and the bridge auditable.
- **Guideline (SDK Contract Discipline):** If a client repo appears awkward because of an existing `Wielder` SDK contract, do not silently reshape the SDK to fit the client. First decide explicitly whether the mismatch is client non-conformance or a real SDK deficiency.
- **Guideline (Sanctioned Framework Change):** If the SDK contract is genuinely suboptimal, raise it as an architectural decision with reasons, tradeoffs, and an improvement plan before changing the shared framework. Unsanctioned SDK drift contaminates every client.
- **Guideline (Client Conformance Default):** If no sanctioned framework change exists, the client repo must conform to the existing SDK contract, even when that contract is inelegant. Adjust local config names, local render logic, docs, and examples in the client rather than moving the shared layer underneath them.
- **Guideline (Native Framework Switching):** When one Wielder entrypoint needs to invoke another entrypoint against a different ecosystem, switch only the required topology dimension through `cli_overrides` and let the downstream entrypoint derive its own resolved `conf`, `action`, and other lifecycle state natively. Do not inject pre-resolved config objects or manually forward derived lifecycle fields when the framework can resolve them itself.
- **Guideline (Topology vs App Contracts):** Strongly suggest using the root/topography accessor only for project-scoped resources and early topology decisions. Do not expect late-derived app/service contracts such as `resolved_images`, resolved service image refs, or `resolved_conf` payload keys from the topography loader. Those belong to `get_app_conf()`, `get_ops_app_conf()`, or `get_service_conf()`.
- **Guideline (Root Ecosystem Purity):** Root ecosystem manifests must stay topological. Do not reference late-derived app/service variables such as `resolved_images`, `resolved_conf`, service image refs, or other app-only payloads inside `conf/ecosystem/<ecosystem>/ecosystem_manifest.conf`. Put those overrides in the app or deploy ecosystem manifest loaded through `get_app_conf()` or `get_service_conf()`.
- **Guideline (No Parallel Config Boot Paths):** Strongly suggest avoiding "side loaders" that separately read `project.conf`, individual ecosystem manifests, or stage manifests just to recreate one app tree. Those parallel config boot paths drift quickly and tend to break when ecosystem families are refactored.
- **Guideline (Notebook Contexts):** Strongly suggest giving notebooks an explicit accessor seam for mode overrides at the same boundary where Wielder normally parses topology, rather than mutating `sys.argv` or trying to inject ecosystem changes after project resolution has already happened.

#### Dogmatic Warning: Gnostic Configs Are Heresy

- **Definition:** A "gnostic config" is any hidden, private, or ad hoc configuration path that bypasses the canonical resolver and silently creates a second source of truth.
- **Forbidden Examples:** new helper-specific HOCON parsers, leaf-script `ConfigFactory.parse_file(...)` calls, one-off deploy-root loaders, silent ecosystem side loaders, or any accessor that exists only to resurrect one narrow config branch.
- **Judgment:** If a script needs configuration, it MUST come through the canonical accessor surface. Do not invent a new priesthood of loaders just because one caller is inconvenient.
- **Purgatory Rule:** If the existing accessor is too narrow, widen the accessor itself carefully. Do NOT create a parallel "temporary" reader. Temporary readers metastasize into permanent architecture rot.
- **Inquisition Rule:** Before adding any new config seam, prove that the existing canonical accessor cannot be extended cleanly. If that proof is absent, the new seam is presumptively invalid.
- **Failure Smell:** If two different code paths can resolve the same ecosystem, app, or deploy contract by reading different file sets, the design is already contaminated.
- **Failure Smell:** If a caller loads a foreign app conf and then falls back to merging it wholesale into the local conf to "make variables appear," the boundary has already dissolved.

### 3. Canonical Structural Mappings over Static Conditionals
A framework built to handle limitless topologies scales significantly better when deferring to explicitly loaded HOCON schemas rather than evaluating static rules.
- **Guideline:** Avoid hard-coding structural environmental identifiers (like `workstation_wsl` or `aws_eks`) directly within Python function logic (e.g., factory if/else statements).
- **Guideline:** Utilize explicit PyHocon mapping registries mapped natively within the project defaults (`e.g., wielder.ecosystem_map`). Use Python `match/case` functional blocks to map the extracted HOCON string cleanly onto an orchestrated class type.
- **Guideline:** When a topological string is unregistered, present the caller with a dedicated `ValueError` advising them to formally declare the new mapping directly in their canonical config tree.

### 4. Semantic Cloud Encapsulation ("Key" vs "Path") & Windows URI Vulnerabilities
In cloud infrastructure, standard POSIX "directory" terminology often implies rigid hierarchical filesystems, whereas blob stores and abstraction layers (S3, Google Drive) operate as flat-key structures.
- **Guideline:** When orchestrating cloud data transfers (like in `Bucketeer`), utilize the terminology `key` (e.g., `object_key`, `dest_key`) rather than `path`. This conceptual shift aligns the tooling seamlessly with the underlying non-hierarchical reality of the storage layer.
- **Guideline (The Windows URI Vulnerability):** You MUST NOT use `os.path.join` to construct Cloud Object Storage URIs or Keys. If a pipeline is executed dynamically on a Windows node natively, `os.path.join` will inject `\` backslashes into the string. S3 and GCP treat `\` as a literal character, not a directory separator, resulting in corrupt, flat blob files (e.g., `raw\protenix\file.json`) instead of physical hierarchical namespaces. ALWAYS use strict POSIX concatenations (e.g., `"/".join(parts)` or `f"{base.rstrip('/')}/{suffix}"`).

### 5. Cross-Topology Flexibility
Core factory functions should support explicit architectural overrides, acknowledging that topologies frequently demand interaction across network boundaries.
- **Guideline:** Expose optional typing identifiers across factory signatures (e.g., `bucketeer_type: str | None = None`). This permits specific components—such as an AWS service needing direct interface with Google Drive—to gracefully bypass standard environment defaults and manually assert their explicitly required target structure dynamically.

### 6. FS Agnostic Storage Boundary Discipline For Data Domains
Filesystem discovery and storage materialization must not leak into DAG logic, business logic, or notebook orchestration simply because the active runtime happens to be local.
- **Guideline:** Any operation that searches storage topology or artifact presence (for example `glob`, recursive file discovery, directory walking, or file existence checks) MUST be routed through a Bucketeer factory method or a domain accessor layered on top of Bucketeer. Application code MUST NOT perform raw OS discovery directly.
- **Guideline:** Any operation that materializes, copies, syncs, or deletes storage artifacts (for example `shutil.copy*`, `os.makedirs`, `os.rename`, `os.remove`, `Path.mkdir`, or ad-hoc directory creation) MUST be owned by Bucketeer or a dedicated accessor, not by the DAG runner.
- **Guideline:** `os.path.join`, `dirname`, and similar functions are prohibited for storage key construction. Parsing a resolved filename token via `basename` or `splitext` is acceptable, but storage lookup and routing are not.
- **Guideline:** Looking up code paths or configuration paths is still path-boundary logic. If code is deciding where config, parquet, images, or outputs live, that resolution belongs in the configuration bootloader, Bucketeer, or a centralized accessor layer.

### 7. Global Stage Tier Nomenclature (`stage_tier`)
Deployment environments (`dev`, `int`, `qa`, `stage`, `prod`) must be physically segregated to prevent data collisions.
- **Guideline:** Use `stage_tier` to define the target environment. This configuration resolves strictly beneath the active `context_conf` pack to guarantee local sandboxes override production defaults.

### 7.1 Central Context Packs over Repo-Local Developer Overlays
- **Guideline:** Standardize developer-local overrides in `context_conf/<name>/` at the super-repo root. Do not keep repo-local `conf/developer/` folders as active peers in the load path.
- **Guideline:** `context_conf/default_conf/` is the canonical baseline context. Additional named packs (for example `context_conf/hermes_batch_qa/`) are encouraged when developers need shareable, versioned local contexts.
- **Guideline:** Inside application repos, track only `conf/context_conf_examples/<name>/`. The live local `conf/context_conf/<name>/` tree is purely local state and must be ignored by Git.
- **Guideline:** The sanctioned operator flow is: copy one example pack, then edit it locally.
  - Example: `cp -r conf/context_conf_examples/default_conf conf/context_conf/default_conf`
- **Guideline:** App-scoped service-shape toggles such as `debug_mode` and `local_mount` should remain app-level config values and be overridden from the active `context_conf` pack, not promoted into a global topological tier.
- **Guideline:** Map physical bucket roots via `stage_tier` (e.g., `starget-<domain>-<stage_tier>`) rather than routing environments manually.
- **Guideline:** Store environment-specific configuration in `starget-data/conf/stage_tier/<stage>/tier.conf` to enforce uniform fallback inheritance.

### 8. Core Topological Dimensions
Agentic configuration reasoning MUST respect the primary topology dimensions engineered into the framework's semantic contract:
- **Ecosystem**: A contextually bound set of distributed surfaces mapped to the workload (e.g., local WSL DAGs synced with Workspace vs AWS Airflow).
- **Compute Surface (`surface`)**: Formally replaces the legacy term `runtime_env`. It strictly defines the material physical execution plane (e.g., `docker`, `kind`, `gcp_gke`). An app-level configuration natively inherits the Ecosystem's default surface but can execute a structural override natively.
- **Stage Tier**: The chronological deployment boundary (`dev`, `stage`, `prod`).
- **Security Mode**: The RBAC and compartmentalization policy (e.g., isolating production secrets via overlapping 'dud-service' architectures).
- **Ephemerality Policy (`deletion`)**: The mandated teardown behavior (`full_ephemeral`, `partial_recycle`, `persistent`). 
- **Deploy Strategy (`canary`)**: The overarching deployment footprint trigger (`standard`, `fuzzy_predicates`).

### 8.1 Service Topology Ownership: Ecosystem over DAG
Distributed service transport selection must ride on the Ecosystem axis, not inside scientific DAG payloads.
- **Guideline:** If an application can operate against multiple transport surfaces (for example `sdk` vs `grpc` for an MSA search dependency), the active mode MUST be resolved from the ecosystem configuration tree, not embedded repeatedly on each DAG target.
- **Guideline:** DAG payloads should declare scientific intent (`use_msa`, upstream dependency presence, target identifiers), not infrastructure reachability modes such as specific local-kube surfaces, cloud providers, or hostnames. Those are environmental topologies and therefore belong to the ecosystem hierarchy.
- **Guideline:** Development ecosystems are allowed to prefer local-first modes (for example `sdk`) even if production ecosystems ultimately prefer remote service transport (`grpc`). This is not wasteful duplication; it preserves local iteration speed and decouples application development from image-build and deploy latency.
- **Guideline:** Container environment variables may mirror topological modes such as `ECOSYSTEM` or `STAGE_TIER` for telemetry and forensic visibility, but application execution logic should prefer resolved configuration trees rather than reading those environment variables as the primary operational truth.
- **Guideline:** Workflow-specific messaging contracts such as Kafka topics and consumer groups should live at ecosystem scope when the workflow is currently 1:1 with that ecosystem. If several ecosystems later share the same workflow contract, extract it into a reusable workflow-level include rather than pushing it back to project root.
- **Guideline (Bind vs Access Ports):** Strongly suggest separating service bind ports from externally consumed access ports. The in-process server bind port belongs to the runtime surface contract, while NodePorts, forwarded ports, or other external access coordinates belong to the deploy/service contract. Do not overload one `grpc_port` field with both meanings.
- **Guideline (No Surface Leak Fixes):** If a deployed service fails due to a port, probe, or container bootstrap mismatch, do not fix it by teaching the application repo about deploy-surface-specific loaders or by making the scientific app reason about Kubernetes particulars. First repair the deploy/service contract and keep the application bound only to a complete ecosystem contract.

### 8.2 Shared Service Layering: Core -> Side-Effect Layer -> Transport
When an application exposes the same capability through both direct API transport and local SDK execution, the implementation must be layered cleanly to avoid WET transport logic.
- **Guideline:** Extract the pure computational core first (for example an MSA search function returning raw `a3m` text).
- **Guideline:** Extract a side-effect layer above that core which owns the canonical external effects such as datalake persistence, manifests, telemetry, and completion markers.
- **Guideline:** The direct API transport layer (for example a gRPC servicer) MUST call the extracted side-effect layer rather than containing unique persistence logic inline.
- **Guideline:** The direct SDK path MUST call that same side-effect layer rather than simulating a local gRPC roundtrip.
- **Guideline:** If a producer DAG, an SDK caller, and a direct API surface such as gRPC all expose the same domain capability, they should converge on the same side-effect layer so that canonical external effects are not fragmented across three implementations.
- **Guideline:** If time and scope were unconstrained, a brokered messaging IO architecture would be the more reactive long-term shape than direct API calls. Until that complexity is justified, keep the present direct API layering explicit and local rather than prematurely splitting into micro-micro services.

### 8.2.1 Runtime Failure Triage: Config Truth vs Live Image Truth
- **Guideline:** Strongly suggest distinguishing three layers during service-failure investigation: resolved repository config, applied deploy manifest, and live container behavior. A match between the first two does not prove the live container is running the intended code.
- **Guideline:** Strongly suggest checking live Kubernetes evidence in this order when a service restarts or fails probes: `kubectl describe`, current logs, previous logs, ConfigMap or Secret payload, and the live Deployment manifest.
- **Guideline:** Strongly suggest treating probe failures and exit code `137` carefully. They often indicate the kubelet killed a process for failing health checks rather than an application exception trace.

### 8.3 Workflow Monitors and Sidecar CLIs
Operational monitors often need both Wielder topology and a small local action vocabulary. That combination should not create a second configuration language.
- **Guideline:** Strongly suggest keeping Wielder topology resolution and script-local actions as separate concerns. The script-local action should be narrow and positional, while ecosystem and stage resolution should still come from the canonical config accessors.
- **Guideline:** Strongly suggest avoiding ad hoc argv rewriting, synthetic parser stacking, or manual flag merging between a local operational CLI and the Wielder CLI surface.
- **Guideline:** When a script has a single dominant interactive behavior such as traffic monitoring, strongly suggest defaulting direct invocation to that behavior and reserving extra actions for explicit internal calls or clearly separated entrypoints.

### 9. The `__file__` Context Drift Antipattern
Legacy python patterns frequently use `os.path.dirname(__file__)` to trace project directories relative to the execution script. In a unified orchestration topology, **this is a catastrophic antipattern**.
- **The Bug:** If a module (like `Wielder`) is globally `pip install`ed, `__file__` will traverse backward into the system Python or PyEnv directories (e.g., `~/.pyenv/versions/...`), completely severing the script from the actual codebase root and corrupting staging directories or Docker contexts.
- **Guideline:** NEVER deduce project geometry or staging roots using `__file__`.
- **Guideline:** There is only ONE source of local Filesystem Truth: **The PyHocon Configuration Stream**. Roots must be strictly evaluated at the configuration source (`conf.super_project_root` or `conf.stage_root`) and statically passed down to orchestrators and SDK execution proxies.

### 10. Inline Import Discipline
Inline imports (import statements nested inside functions or methods) obscure dependency trees, trigger unpredictable parsing latency during execution sequences, and fracture static analysis graphs.
- **Guideline:** NEVER use inline imports natively within Wielder execution graphs or applications. ALL imports must be physically hoisted to the global `HEAD` of the module.
- **Exceptions:** Inline imports are strictly forbidden unless mathematically necessary to break confirmed circular dependencies, isolate heavily localized Process/Thread initializations, or conditionally load massive multi-gigabyte Data Lake libraries (e.g., `PyTorch` or `Tensorflow` in a lightweight REST API). Short of those exact hardware/memory constraints, use global imports natively.
