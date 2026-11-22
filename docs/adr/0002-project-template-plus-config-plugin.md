---
status: accepted
date: 2026-11-21
---

# The harness ships as two artifacts: a project template and a configuration plugin

Guardrails sit at two layers, and each layer has its own distribution model.

**The portable layer goes in a project template.** This layer is copied into each project once, and the project then owns its copy. It includes:

- CI pipelines;
- configuration for tests, static analysis and formatting;
- git hooks;
- the test-lock and mutation Gates.

CI runs this layer inside the project, so it has to live there. It also works with any agent runtime, or with none.

**The runtime layer goes in a separate configuration repository, published as a Claude Code plugin.** It is enabled per project and versioned. It includes:

- skills, including the TDD workflow skill;
- Claude Code tool hooks;
- permission settings.

A template alone drifts: every project ends up with its own outdated copy of the skills. A loaded configuration alone cannot supply CI and Gate files, which must sit inside the project.

## Considered options

- **Template only.** This was the original `agentic-scaffold` idea.
  - Rejected: runtime configuration copied into each project drifts and cannot be improved in one place.
- **Symlink installer at user scope.** This is the pattern of the employer's `agent-config`.
  - **For:** simple, and `git pull` updates it instantly.
  - **Against:** it applies the same configuration to every project on the machine, but a Laravel project and a TypeScript project need different Gates and skills.
  - Rejected.
- **A runtime abstraction that compiles to several agent tools.** Rejected: the portable layer already provides runtime independence (hypothesis H4), at no extra cost.

## Consequences

- The plugin is specific to Claude Code. Moving to another runtime means rewriting only the thin runtime layer.
- Changes to the template do not reach existing projects automatically. That is acceptable, because projects own their CI.
