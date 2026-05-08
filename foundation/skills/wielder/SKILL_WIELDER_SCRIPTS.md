---
name: Wielder Scripting & Evaluation Skills
description: Core architectural patterns for writing `Wielder` orchestration and evaluation scripts, focusing on configuration boundaries, dependency inheritance, and physical formatting conventions.
---

# Wielder Script Architectural Patterns

When writing or modifying `Wielder` orchestration scripts (e.g., K8s deployment wrappers, `bootstrap.py` sequences, or local pipeline evaluators), we generally adhere to the following `Wielder-Antifragile` guidelines to help them scale gracefully across environments. 

## 1. Avoid Reinventing the Wheel
`Wielder` exists as a specialized ecosystem library to handle orchestration heavy lifting. As a rule of thumb, scripts work best when they act as thin, declarative wrappers linking `Wielder` primitives to execution targets. 

If you find yourself writing custom logic to sync directories, parse YAML text, or read `.env` configuration files manually, it usually implies there's a better path:
* Look for established implementations natively within `Wielder` utility modules (`wielder.util`, `wielder.wield.project`, etc.). 
* Depend natively on built-in tools like `KubectlBucketeer`, native PyHocon parsers (`get_app_conf()`), and native deployment classes.

## 2. Configuration-Driven Filesystem Operations
Wielder orchestration scripts shy away from hardcoded directory paths, deeply nested string concatenations, or magic constants pointing to files.

* The preferred way a script relates to the virtual or physical file system is through the centralized Configuration (PyHocon `.conf` files).
* Utilize `get_app_conf()` or native Wielder extraction patterns to determine the contextual `staging_root`, `bucket_path`, or `target_database` dynamically.
* When local development context matters, scripts should rely on the active central `context_conf/<name>/developer.conf` pack rather than repo-local `conf/developer/` overrides or ad hoc CLI flags.
* For app-local examples, keep tracked packs under `conf/context_conf_examples/<name>/` and keep `conf/context_conf/<name>/` ignored. The human and agent workflow is always: copy an example pack into `conf/context_conf/`, then edit the copied local pack.

## 2.1 Thin Script, Single Loader
Operational scripts become fragile when they grow a second understanding of configuration structure.

* Strongly suggest resolving application state through the canonical accessor that matches the target domain, rather than re-merging `project.conf`, `ecosystem_manifest.conf`, and `stage_tier` files by hand.
* Strongly suggest keeping the script thin enough that it owns execution sequencing, not config reconstruction.
* Strongly suggest avoiding local parser inventions that partially duplicate Wielder behavior, because those forks tend to drift exactly when ecosystem naming or family extraction changes.
* When a script must orchestrate a child repo, strongly suggest loading that child repo through the child repo's own canonical accessor rather than inventing a second local reader.
* Strongly suggest treating such cross-repo access as explicit dependency wiring, not as a generic framework feature. The script should read foreign owned fields, not absorb the foreign app's whole config identity.
* If the bridge logic is only a few lines, keep it WET and local on purpose. A garden of tiny explicit bridges is healthier than a premature generic loader that hides ownership.

## 2.2 Local Script Actions vs Wielder Modes
Some scripts need a small local command vocabulary in addition to Wielder topology.

* Strongly suggest keeping those local actions narrow and positional while leaving ecosystem and stage resolution to the normal Wielder/config accessor path.
* Strongly suggest defaulting direct CLI execution to the dominant operational behavior when one obviously exists, rather than multiplexing many loosely maintained modes through one argv surface.
* Strongly suggest separating internal programmatic action hooks from human-facing shell invocation if a script starts accumulating too many modes.

## 2.3 Granular Apply/Delete Control
Deployment workflows frequently need asymmetric behavior between `apply` and `delete`. A step that is desirable during bring-up is often dangerous or wasteful during teardown.

* Strongly suggest keeping `apply`/bring-up controls in a dedicated `deploy_steps` block and `delete`/teardown controls in a separate `delete_steps` block rather than reusing one boolean family for both directions.
* Strongly suggest making delete behavior explicitly voidable at the config layer. If a workflow should preserve third-party services, topics, port-forwards, or infrastructure during delete, that decision should be expressed in `delete_steps`, not hardcoded in the script.
* Strongly suggest letting the script branch on `WieldAction` and then read the matching config family, rather than treating `delete` as a blind inversion of `apply`.
* When a deploy script orchestrates several resources, strongly suggest exposing delete granularity per resource class such as third-party services, topics, local port-forwards, workload services, and infrastructure.

## 2.4 Long-Running Operator Handoff
Some Wielder actions are expected to run for minutes or hours, especially bucket mirrors, storage syncs, image builds, Terraform applies, large data ingestion jobs, and cloud workflow executions.

* Agents should run short validation actions themselves, such as `show`, `plan`, `probe`, linting, focused tests, and command construction checks.
* Agents should not start long-running `apply`, sync, mirror, clone, build, or migration jobs unless the operator explicitly asks the agent to run them and wait.
* For long-running jobs, provide the exact absolute command for the operator's terminal, including the repository-root-safe script path and required Wielder modes.
* Before handing off, verify that config resolves and that the command shape is correct with the lightest available action (`show`, `plan`, or `probe`).
* After handoff, treat pasted terminal output as the continuation point. Diagnose failures from that output and patch the smallest relevant source/config boundary.
* If an agent accidentally starts a long-running local process and it is not needed for immediate inspection, stop it cleanly when safe and give the operator the command to rerun.

## 2.5 Provision Runtime Tooling From Config
When a Wielder script depends on a backend CLI, the script should remove manual setup burden where it can do so safely.

* Prefer generating or ensuring local CLI configuration from the resolved Wielder config before asking the operator to enter an interactive setup flow.
* Keep generated local configuration in developer-local paths, usually under `~/.config/<tool>/`, and point to that path from `context_conf/<name>/developer.conf`.
* For clone backend configuration, prefer a generic WCloner-side config factory such as `WCloner.configure_wclone(...)` that materializes all configured remotes from HOCON before planning or execution.
* Do not version secrets, OAuth tokens, access keys, or one-off migration coordinates. Version only the reusable config shape and examples.
* If a CLI can consume short-lived credentials through environment variables, inject them at execution time rather than materializing tokens into versioned config.
* For GCP-hosted execution, prefer service-account ADC or metadata credentials with backend `env_auth = true`; do not design hosted jobs around interactive `gcloud` OAuth or operator-local tokens.
* For local GCP-dev execution, short-lived `gcloud auth print-access-token` injection is acceptable when it avoids writing tokens into WClone backend config.
* Preserve provider surfaces through factories and accessors. A script should ask a Bucketeer/WCloner-style surface to ensure a bucket or destination, not branch on concrete provider names except at the factory registry boundary.
* Add new reusable capabilities to the Wieldable Functionalities catalog when they become stable operator-facing patterns.

## 3. Standalone Invocation Formatting
Because Wielder evaluation scripts are often automated or executed directly across various shells (WSL, native Linux, CI/CD), they benefit from secure formatting to prevent common shell evaluation traps (e.g., the ImageMagick `import` bash hijacking).

To avoid silent execution hangs, a standard `*.py` script intended for execution (e.g., containing an `if __name__ == '__main__':` block) typically implements two straightforward conditions:

1. **The Unix Shebang**:
   The first line of the evaluation script designates the Python environment to bypass default Bash interpretation:
   ```python
   #!/usr/bin/env python
   ```
2. **Executable Permissions**:
   The script is granted explicit execution rights at creation:
   ```bash
   chmod +x <filename>.py
   ```

By adhering to this pattern, cross-boundary invocations like `./fetch_protenix_bundle.py` or `./sanity.py` are free to execute natively as Python processes without relying on the shell's fallback assumptions.
