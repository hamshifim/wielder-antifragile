---
description: Wielder PyHocon Configuration Guidelines (Strict Architectural SOP)
---

# Wielder PyHocon Configuration Guidelines

The Wielder framework relies on a deterministic and structurally sound configuration layer. As data domains and topologies scale, adherence to these engineering constraints improves maintainability, preserves clear failure boundaries, and supports consistent structural abstraction.

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

5. **Developer Overlays (`developer.conf`)**
   * **Role**: Purely localized, untracked evaluations dictating aggressive isolation boundaries dynamically during active development. Trumps all downstream domain boundaries.

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
- **Guideline:** Use `stage_tier` to define the target environment. This configuration resolves strictly beneath `developer.conf` to guarantee local sandboxes override production defaults.
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

### 9. The `__file__` Context Drift Antipattern
Legacy python patterns frequently use `os.path.dirname(__file__)` to trace project directories relative to the execution script. In a unified orchestration topology, **this is a catastrophic antipattern**.
- **The Bug:** If a module (like `Wielder`) is globally `pip install`ed, `__file__` will traverse backward into the system Python or PyEnv directories (e.g., `~/.pyenv/versions/...`), completely severing the script from the actual codebase root and corrupting staging directories or Docker contexts.
- **Guideline:** NEVER deduce project geometry or staging roots using `__file__`.
- **Guideline:** There is only ONE source of local Filesystem Truth: **The PyHocon Configuration Stream**. Roots must be strictly evaluated at the configuration source (`conf.super_project_root` or `conf.stage_root`) and statically passed down to orchestrators and SDK execution proxies.

### 10. Inline Import Discipline
Inline imports (import statements nested inside functions or methods) obscure dependency trees, trigger unpredictable parsing latency during execution sequences, and fracture static analysis graphs.
- **Guideline:** NEVER use inline imports natively within Wielder execution graphs or applications. ALL imports must be physically hoisted to the global `HEAD` of the module.
- **Exceptions:** Inline imports are strictly forbidden unless mathematically necessary to break confirmed circular dependencies, isolate heavily localized Process/Thread initializations, or conditionally load massive multi-gigabyte Data Lake libraries (e.g., `PyTorch` or `Tensorflow` in a lightweight REST API). Short of those exact hardware/memory constraints, use global imports natively.
