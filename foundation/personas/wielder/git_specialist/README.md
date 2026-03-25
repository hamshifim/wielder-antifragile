# Git Specialist Bundle

This directory is the first exemplar of the object bundle convention.

- [persona.hocon](/home/gideon/starget/wielder-antifragile/foundation/personas/wielder/git_specialist/persona.hocon)
  Structural metadata, inheritance, tools, outputs, and final fields.
- [body.md](/home/gideon/starget/wielder-antifragile/foundation/personas/wielder/git_specialist/body.md)
  Human-authored markdown body for the persona.

The intended wrapper behavior is:

1. load `persona.hocon`
2. resolve inheritance
3. load `body.md`
4. render a resolved audit artifact into `.foundation_audit/`

