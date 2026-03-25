---
description: Wielder PyHocon Configuration Guidelines (Strict Architectural SOP)
---

# Wielder PyHocon Configuration Guidelines

The Wielder framework relies on a deterministic and structurally sound configuration layer. As data domains and topologies scale, adherence to these engineering constraints improves maintainability, preserves clear failure boundaries, and supports consistent structural abstraction.

### 1. Strict Attribute Resolution
Configuration dependencies should guide the execution toward immediate awareness if environment parameters are missing.
- **Guideline:** Avoid wrapping configuration extraction blocks in generic `try...except Exception` silencing patterns. 
- **Guideline:** Prefer extracting properties natively via dot-notation (e.g., `conf.ecosystem`). Allow PyHocon to natively trigger a `ConfigMissingException` or `AttributeError` trace upon failure. Suppressing missing configurations obscures orchestration defects.

### 2. Guardrails Against Rogue CLI and OS Mappings
Wielder possesses a fledgling CLI, and other external interfaces may be actively developed in the future. However, it is a strict guardrail—particularly for AI agents—to avoid flooding the core config components with fragile system calls or ad-hoc argument parsers.
- **Guideline:** Minimize the direct instantiation of `argparse` bindings or manual `os.environ` queries embedded actively within core PyHocon cascade logic (e.g., `wield_conf.py`).
- **Guideline:** External coordinates should ideally be injected into the config hierarchy as native PyHocon dictionary fallbacks natively upstream. Let PyHocon organically orchestrate dynamic variable interpolation via `.with_fallback(..., resolve=True)`.

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
