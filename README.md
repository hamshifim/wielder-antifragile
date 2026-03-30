# wielder-antifragile

`wielder-antifragile` is the doctrinal and governance layer for wielding complex distributed systems.
It defines how agentic work is structured, inherited, validated, and audited.

## What This Repo Is

- Canonical home for personas, skills, workflows, contracts, and playbooks.
- Source-truth semantic bundles (`*.hocon` + `body.md`) that resolve into auditable artifacts.
- Strict provenance and execution-identity policy for multi-agent environments.

## What This Repo Is Not

- It is not the runtime infra executor.
- It is not coupled to a single runtime implementation.

`Wielder` (the repo/toolkit) is one runtime implementation that can satisfy these contracts, alongside other provider/tooling stacks.

## Current Architectural Position (Phase 0)

- Keep adoption modular: projects consume `wielder-antifragile` as a submodule.
- Preserve flexible usage but enforce strict contracts.
- Use single-parent persona inheritance via `base`.
- Reject non-empty `extends` in persona resolution.
- Enforce explicit override allowlists and inherited `final` constraints.

## Execution Identity Contract

Execution identity is mandatory under `wielder_antifragile.execution_identity` and must include:

- `user_name`
- `provider`
- `agent_type`
- `agent_instance_id`
- `agent_name`
- `agent_version`
- `session_id`
- `run_id`
- `invoked_at_utc`

Canonical naming:

- `agent_name = <provider>_<agent_type>_<agent_instance_id>`
- Example: `openai_codex-cli_1`

Config loading fails explicitly if any required field is missing/empty or if `agent_name` does not match the canonical composition.

## Repository Map

```text
wielder-antifragile/
  conf/
  docs/
    antipatterns/
    ephemeral-super-cluster-wielder.md
    foundation-manifesto.md
    implementation-architecture.md
  foundation/
    personas/
    skills/
    workflows/
    contracts/
    playbooks/
  src/wielder_antifragile/
    core/
    personas/
  tests/
  .foundation_audit/
```

## Quick Validation

Run from `wielder-antifragile/`:

```bash
PYTHONPATH=src pytest -q tests
```

Smoke-check persona resolution:

```bash
PYTHONPATH=src python -c "from wielder_antifragile.personas.object import PersonaObject; print(PersonaObject.from_id('git_specialist').to_string())"
```

## Primary Docs

- [Foundation Manifesto](docs/foundation-manifesto.md)
- [Implementation Architecture](docs/implementation-architecture.md)
- [Ephemeral Super Cluster Architecture](docs/ephemeral-super-cluster-wielder.md)

## Agentic Antipatterns

As autonomous orchestration scales, Wielder explicitly defends against mathematical regressions.
- [Context Drift (Fantasia Risk)](docs/antipatterns/context-drift.md): Structural defense against high-velocity scope hallucination.
- [Style Drift (The OOD Paradox)](docs/antipatterns/style-drift.md): Structural defense against agents unconsciously regressing idiosyncratic code towards training-set norms.
- [Hard-Stopping (The Security Straitjacket)](docs/antipatterns/hard-stopping.md): Structural critique of stunting agentic growth through permanent action bans instead of explicit skill wrappers.
