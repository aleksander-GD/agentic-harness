---
status: accepted
date: 2026-11-22
---

# v1 Runs are plugin workflows that the owner starts, paid for by the subscription

In v1, the owner starts every Run inside Claude Code. A Run is made of two Claude Code workflows shipped in the harness plugin:

- `plan`: turns a Spec or a prompt into a Plan, then stops;
- `build`: Red Commit → green → Gates → review → pull request.

Between those two stops the Run continues on its own. State between the workflows lives in files and git. That keeps ADR-0004's design, a deterministic orchestrator with state in files, but uses Claude Code's own workflow engine in place of a separate command-line tool.

The `build` workflow runs in the devcontainer, with the Claude Code login kept on a mounted volume, as C16 requires.

Visibility comes from Claude Code itself:

- agent view;
- `/workflows`;
- `Notification` hooks;
- optionally, a mod pane.

## Why

The subscription (C22) is the owner's budget. Anthropic's Consumer Terms bar scripted, non-human access except through an API key or with explicit permission. A command-line tool calling `claude -p` repeatedly and unattended on a subscription would rest on an unresolved reading of those terms.

When the owner starts each Run in the app, the harness stays within ordinary interactive use. It also gets native visibility, and it matches the plugin-based approach a colleague already uses in practice.

## Considered options

- **A TypeScript command-line tool calling `claude -p` per stage on the subscription.** This was C33.
  - **For:** headless and easy to test.
  - **Against:** relies on a doubtful reading of the terms. `--bare` is due to become the default and would drop the subscription login.
  - **Rejected.**
- **API-key runs, fully unattended**, from a command-line tool or GitHub Actions.
  - **For:** clearly permitted, and a Run can start without the owner.
  - **Against:** every token costs money, which reopens model routing (C23).
  - **Deferred.** It is the upgrade path for when Runs should start without the owner.

## Consequences

- Upgrading to unattended Runs later means adding a new way to *start* a Run. The plugin, skills, Gates and file-based state stay the same.
- Workflows cannot pause in the middle, so every point where the owner must approve something has to fall on a boundary between workflows.
- Facts to check while building:
  - **Plugin sub-agents.** Sub-agents shipped in a plugin ignore `permissionMode` and `hooks`. Any guardrail on them must come from settings or the sandbox, not from the agent definition.
  - **Agent view and the devcontainer.** It is not yet confirmed whether agent view and desktop notifications see a CLI session running inside the devcontainer.
  - **Plugin settings.** It is not yet confirmed whether a plugin can ship sandbox and permission settings.
