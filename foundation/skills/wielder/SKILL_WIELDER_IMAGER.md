---
name: "Wielder Imager & Staging Sandboxing"
description: "Core architectural mandate regarding Docker context execution, outlining the necessity of the isolated 'Staging Sandbox' pattern, Naked Execution Wrappers, and unified Naming Conventions."
---

# Wielder Imager Staging Mechanism

## The Docker Daemon Antipattern
A common developmental antipattern is attempting to execute `docker build -f <Dockerfile> .` directly against the live Git super-repo source (the "Super Project Root").

While Docker natively handles context tarballing, executing this against live, massive super-repos creates critical structural failures:
1. **IDE Throttling & Traffic Lockups:** When Docker parses a giant, multi-asset context, local IDEs drastically spike CPU/IO attempting to track the simultaneous read operations, locking the developer out of simultaneous coding.
2. **Third-Party Asset Pollution:** Often, Docker builds require injecting arbitrary external weights, `.env` files, or binaries. If the build context is the live repository, these assets must be generated directly inside the tracked tree, aggressively polluting the pristine Git structure.
3. **Temporal Smearing:** A 10-minute Docker compilation reading from a live, mutable directory risks pulling in partial file states if the developer modifies the code mid-build.

## The Antifragile Solution: Staging Sandboxes
To resolve this, the `Wielder` Imager (`pack_image_antifragile`) strictly mandates the use of an isolated **Staging Sandbox**.

### Execution Rules
1. **Snapshotting:** Before `docker build` is called, a fast `shutil.copytree` (ignoring `.git`, `__pycache__`, and IDE `.idea` bloat) replicates the active, necessary modules into a UUID-isolated `/artifacts` temporary directory.
2. **Committed Super-Repo Determinism:** In a Git super-repo with submodules, image baking must be treated as a committed-state operation keyed by one final committed super-repo SHA. If the image is sourced through the super-repo tree, the bake reflects the committed submodule revisions captured by that super-repo SHA, not merely the editor buffer.
3. **Mandatory Bake Order for Submodules:** If a deployment depends on changes inside a submodule (for example `starget-in-silico`), the correct sequence is: commit the submodule changes, commit the super-repo pointer update, then bake and push the image, then deploy. Failing to complete this chain yields stale configs or stale code inside the container even when the local workspace looks correct.
4. **Decoupling:** Because the staging environment is instantly decoupled from the live code, the developer can instantly return to writing code on the live footprint. The 10-minute Docker daemon build operates silently against the staging clone.
5. **Live Runtime Verification over Assumption:** A successful config render or `kubectl apply` is not proof that the new image behavior is live. If the image tag is unchanged or the staged content still points at committed state, the running container may still reflect older code. Verify with live pod logs and rollout state rather than assuming the workspace diff reached the container.

## Workflow-Driven Image Verification
For workflow entrypoints that both build images and deploy them, the most reliable integration check is the exact workflow itself rather than a disconnected sequence of helper invocations.

For the broader doctrine that treats Wielder workflows as configurable integration, system, load, and production execution surfaces across ecosystems and stage tiers, see [Workflow Validation Guidelines](file:///home/gideon/starget/wielder-antifragile/foundation/skills/wielder/SKILL_WORKFLOW_VALIDATION_GUIDELINES.md).

- **Rule:** The workflow's own `delete -> apply` cycle is the correct integration harness when validating image-bearing changes.
- **Rule:** The deployment identity is the final committed super-repo SHA. Build, push, and deploy must all resolve against that same SHA.
- **Rule:** If a thin deploy-orchestrator repository is not itself part of the baked image, treat its local diff as deployment wiring rather than image truth, and do not confuse those two roles during validation.
- **Rule:** Keep `imagePullPolicy: Always` on these workflow-managed integration deployments so the deployment actually tests the image that was just baked and pushed.
- **Rule:** When a workflow `apply` both bakes and deploys, a successful end-to-end `apply` is the primary integration and system proof for that image delta.
- **Rule:** When a rollout fails after an apparently correct config change, compare the resolved config, the applied Deployment manifest, and the live pod log. If the live pod behavior still matches the old code path, suspect stale committed image content before inventing new topology hacks.

## Explicit Topology over Internal Context
The sandbox path (e.g., `~/artifacts/starget_base...`) must NEVER be inferred inside the `pack_image` executors using legacy methods like `__file__` or `get_super_project_roots()`.
The topographical source of truth is exclusively the **PyHocon Configuration Stream**. The Caller Execution Script (`image.py`) securely evaluates the PyHocon context natively and explicitly passes the `conf.super_project_root` into the imager proxy as `staging_root`. This completely centralizes the architectural footprint natively into the PyHocon boundaries, ensuring zero Context Drift for globally installed packages.

---

# Image Modulation & Topographical Modes
To ensure images seamlessly adapt to the surfaces they land on, Docker tags strictly obey native PyHocon runtime boundaries.

### 1. Dynamic Mode Tagging
Container tags must fundamentally flex against the terminal topology (e.g., inheriting `stage_tier` to dynamically shift between `:dev` and `:prod`).
- **Rule:** Project-level files (`image.conf`) define generic strings (`base_image_name = "starget/starget_base"`). The dynamic combination strictly occurs via string interpolation: `base_image_full = ${base_image_name}":"${stage_tier}`.

### 2. App-Level Reference Pointers
When an internal Application builds on a platform Base Image, it mathematically binds to it by overriding the local PyHocon array using project inheritance.
- **Rule:** Do not hardcode project image arrays into App `Dockerfiles`. Instead, the Application explicitly declares its dependency inside `app.conf` (e.g. `base_image = ${base_image_full}`).

### 3. Agnostic Docker Contexts
- **Rule:** Target Dockerfiles must replace hardcoded `FROM` repository targets with naked argument variables (e.g., `ARG BASE_IMAGE` `FROM ${BASE_IMAGE}`). The orchestrator (`imager.py`) seamlessly yields the dynamically requested property via the `--build-arg BASE_IMAGE=` payload.

### 4. Runtime Mode Propagation
- **Rule:** It is acceptable and often useful to mirror resolved topological modes such as `ECOSYSTEM` and `STAGE_TIER` into container environment variables for telemetry, inspection, and forensic replay.
- **Rule:** Those environment variables should not become the preferred execution source of truth if the workflow can instead stage a native configuration artifact such as `context_conf/<name>/developer.conf` into the runtime surface.
- **Rule:** For workflow ecosystems, prefer copying the staged configuration artifact after clone/update over permanently depending on deploy-time CLI mode injection inside container commands.

### 5. Foreign-Owned Image Contracts
- **Rule:** If an image wrapper in the orchestrating repo bakes code owned by a child repo, the image metadata and docker context path should come from the child repo's canonical app conf, not from ad hoc local reconstruction.
- **Rule:** The orchestrator conf and the foreign image-owner conf serve different roles. Keep execution identity, topology, and tagging in the local caller conf; keep image-owned fields such as `dockerfile_dir`, `repository_name`, and base-image relationships in the foreign owner conf.
- **Rule:** Do not merge the entire foreign app conf into the local execution conf to make image fields "available." Read the foreign owned image subtree and pass it explicitly into the imager seam.
- **Rule:** If each `<app>_image.py` already knows which foreign app it wields, let that leaf module import the foreign repo's canonical accessor directly and keep the bridge thin and explicit.

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
