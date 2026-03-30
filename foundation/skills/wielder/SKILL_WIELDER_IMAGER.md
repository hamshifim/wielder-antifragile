---
name: "Wielder Imager & Staging Sandboxing"
description: "Core architectural mandate regarding Docker context execution, outlining the necessity of the isolated 'Staging Sandbox' pattern, Naked Execution Wrappers, and unified Naming Conventions."
---

# Wielder Imager Staging Mechanism

## The Docker Daemon Antipattern
A common developmental antipattern is attempting to execute `docker build -f <Dockerfile> .` directly against the live Git monorepo source (the "Super Project Root").

While Docker natively handles context tarballing, executing this against live, massive monorepos creates critical structural failures:
1. **IDE Throttling & Traffic Lockups:** When Docker parses a giant, multi-asset context, local IDEs drastically spike CPU/IO attempting to track the simultaneous read operations, locking the developer out of simultaneous coding.
2. **Third-Party Asset Pollution:** Often, Docker builds require injecting arbitrary external weights, `.env` files, or binaries. If the build context is the live repository, these assets must be generated directly inside the tracked tree, aggressively polluting the pristine Git structure.
3. **Temporal Smearing:** A 10-minute Docker compilation reading from a live, mutable directory risks pulling in partial file states if the developer modifies the code mid-build.

## The Antifragile Solution: Staging Sandboxes
To resolve this, the `Wielder` Imager (`pack_image_antifragile`) strictly mandates the use of an isolated **Staging Sandbox**.

### Execution Rules
1. **Snapshotting:** Before `docker build` is called, a fast `shutil.copytree` (ignoring `.git`, `__pycache__`, and IDE `.idea` bloat) replicates the active, necessary modules into a UUID-isolated `/artifacts` temporary directory.
2. **Capturing Uncommitted State:** The copy instantly bridges any uncommitted working changes without relying on a `WGit` pipeline, enabling true rapid-iteration development.
3. **Decoupling:** Because the staging environment is instantly decoupled from the live code, the developer can instantly return to writing code on the live footprint. The 10-minute Docker daemon build operates silently against the staging clone.

## Explicit Topology over Internal Context
The sandbox path (e.g., `~/artifacts/starget_base...`) must NEVER be hardcoded inside the `pack_image` executors.
The topographical source of truth is always the **Caller Execution Script** (`image.py`). The caller securely evaluates the PyHocon context (via `get_super_project_roots()`) and explicitly passes the `staging_root` into the imager proxy. This completely centralizes the architectural footprint natively into the deployment configuration boundaries, ensuring zero context drift across K8s tiers.

---

# Wielder `image.py` Naming Convention

## Core Principle
Under the unified Wielder orchestration framework, deploying containers via `wielder.util.imager` relies on strict architectural rigidity. 

**All image wrapper scripts must be uniformly named `image.py`.**
Do not suffix the script Name (e.g., `bootcassandra_image.py` or `msa_inference_server_image.py` are explicitly prohibited). 

## Identity via Fully Qualified Path
Because all scripts are identically named, their **identity is purely derived from their fully qualified path**. The script's unique context is parsed through the nested directory structure:

### Standard Module Topology
`apps/<app_name>/deploy/<module_name>/wield/image.py` OR `apps/<app_name>/image.py`

By enforcing standard filenames, CI/CD traversal scripts, execution bot runners, and orchestration linters do not require magic string interpolation or regex hacks. They can simply search for `image.py` under the target module's deployment namespace to find the absolute entrypoint. 

---

# Wielder Execution Wrapper Guidelines

The Wielder architecture physically strips localized operations logic away from the naked execution wrappers. Because these scripts act as literal execution proxies across CLI interfaces and CI/CD pipelines, they must strictly adhere to Native POSIX Execution Standards.

### 1. The Shebang Mandate
Every execution wrapper acting as an entry-point (e.g., `image.py`) MUST physically declare the absolute Python environment path at line 1.
- **Guideline:** Begin every orchestrating script with `#!/usr/bin/env python`.
- **Reasoning:** This structurally ensures that the pipeline honors the exact `pyenv` or active virtual environment natively engaged in the terminal context, mapping Python physically without relying completely on the `python` bash keyword.

### 2. Physical Executable Permissions
A script cannot act as an autonomous execution proxy if the filesystem boundary natively blocks it.
- **Guideline:** All deployment scripts MUST be explicitly granted execution permissions (`chmod +x`).
- **Reasoning:** During automated CI/CD runs (e.g., GitHub Actions, Jenkins), or when interacting recursively over Wielder's native execution loops, failure to structurally embed `+x` throws Permission Denied `126` crashes.

### 3. Execution Standard Example
A formally compliant script physically manifests entirely naked logic alongside pure execution capability:

```python
#!/usr/bin/env python
import logging
from wielder.util.log_util import setup_logging
from starget_wielder.deploy.imager import pack_starget_image

def msa_inference_server_image():
    pack_starget_image(app_name="msa_inference_server")

if __name__ == "__main__":
    setup_logging(log_level=logging.DEBUG)
    msa_inference_server_image()
```
*Note that the function explicitly defines its context, the shebang routes Python automatically, and the file acts natively as an executable.*
