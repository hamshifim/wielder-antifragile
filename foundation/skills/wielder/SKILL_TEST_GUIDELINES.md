---
description: Architectural Testing & Live QA Execution Protocols
---
# Testing Constraints & Live QA Protocols

- **The Live Execution Rule**: Tests assessing integration interfaces, Data Lake endpoints, or stochastic Large Language Models (LLMs) MUST inherently execute live data schemas. You MUST NOT use `unittest.mock` to artificially spoof remote model APIs or PySpark IO logic. If it interacts with reality, you verify reality.
- **Physical Sandboxing**: Live tests that write physical data arrays MUST securely map outputs strictly to ignored `/tmp/` artifacts or natively `.gitignore`'d Sandbox outputs (e.g. `explore_output/`) to explicitly prevent committing multi-megabyte artifact bloat into the git history.
- **Abstract Function Independence**: Tests MUST target pure leaf nodes iteratively. Assess module functions by feeding them strictly primitive independent datatypes (URI strings, integer dimensions, explicit configuration dictionaries) rather than demanding massive monolithic DAG Orchestration boots to execute a trivial test slice.
- **Analytical Pruning Parity**: When testing PySpark architectures, assertion testing MUST concretely validate that O(1) query-pruning logic functions correctly against `<global_conf_id>` dimension boundaries, proving the pipeline bypasses irrelevant namespaces.
