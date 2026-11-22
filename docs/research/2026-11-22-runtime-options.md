# Runtime options for autonomous Runs: headless, plugin, workflows

- **Date:** 2026-11-22
- **Sources:** official docs under https://code.claude.com/docs/en/, read on that date, and Anthropic's Consumer Terms (https://www.anthropic.com/legal/consumer-terms).
- **Status:** research input for T7.

Each statement carries a tag:

- **[documented]**: the source says it.
- **[inference]**: our own interpretation.
- **[unresolved]**: neither the docs nor the terms settle it.

## Authentication and terms

- By default, `claude -p` uses the subscription login. [documented: headless]
- `--bare` ignores that login and needs an API key. The docs say `--bare` "will become the default for -p in a future release". [documented: headless]
- `claude setup-token` creates a one-year subscription token for CI and scripts, used as `CLAUDE_CODE_OAUTH_TOKEN`. The GitHub Actions docs accept this token. [documented: authentication, github-actions]
- The Consumer Terms, section 3, bar access "through automated or non-human means, whether through a bot, script, or otherwise". The exceptions are access through an API key, or where Anthropic explicitly permits it. [documented: consumer-terms]
- Subscription login is described as "designed to support ordinary use of Claude Code". [documented: legal-and-compliance]
- **Is it permitted to run `claude -p` unattended and repeatedly on a subscription?** [unresolved]
  - The docs support subscription tokens for CI.
  - The terms restrict scripted access.
  - An API key is the path that is clearly allowed.
- The Agent SDK quickstart assumes an API key. Whether a solo developer may run SDK scripts on a subscription login is not documented. [unresolved]

## Workflows

- **What they are.** A workflow is a JavaScript script that orchestrates sub-agents. It lives in `.claude/workflows/` or `~/.claude/workflows/`, or ships inside a plugin's `workflows/` folder. It is watched with `/workflows`. [documented: workflows]
- **No human input during a run.** For sign-off points, the docs recommend splitting the work into separate workflows. [documented]
- **Resuming.** Resume works only within the same session. A fresh session starts the workflow from the beginning. [documented]
- **Fit with this design.** A Run's two stops (C29) fall exactly on the boundaries between workflows: a *plan* workflow, then a *build* workflow. State in between lives in files and git, which matches ADR-0004. [inference]

## What a plugin can contain

**Contents.** [documented: plugins/components]

- skills and commands;
- agents;
- hooks;
- MCP and LSP servers;
- output styles and themes;
- monitors;
- workflows;
- `bin/` executables;
- mods.

**Limits on sub-agents shipped in a plugin.** They ignore `permissionMode`, `hooks` and `mcpServers`. [documented]

**Mods.** A mod is a JavaScript handler running inside Claude Code. [documented: mods/overview]

- **What it can do:**
  - draw panes and bands, and add commands;
  - hold or rewrite tool calls;
  - approve tool calls that the user's rules would otherwise block.
- **Where it runs:** the terminal, the Desktop app's Code tab, and VS Code. It does **not** run under `-p` or the SDK.
- **Security note.** [inference] A mod that can approve blocked tool calls is trusted code with power over the permission system. Read it before installing it.

**Status line in a plugin.** Whether a plugin can ship a status line is not documented. [unresolved]

## Visibility and notifications

- **Agent view** shows each session's state: Working, Needs input, Idle, Completed, Failed, Stopped. Sessions that need input are listed first. `claude agents --json` exposes the state, including what a session is waiting for. [documented: agent-view]
- **The `Notification` hook** fires on permission prompts, idle prompts and similar events. [documented: hooks]
- **The desktop app** sends an OS notification when a session finishes and the user is not viewing it. [documented: desktop]

## Sub-agents

- A sub-agent definition can set its own model, tools, permissions, maximum number of turns, skills and isolation. [documented: sub-agents]
- Each sub-agent starts with a fresh context. [documented: sub-agents]

## Devcontainer authentication

- The login persists only if `~/.claude` is mounted as a volume. A subscription token can also be supplied as an environment variable. [documented: devcontainer]
- If permission checks are skipped, a malicious repository can exfiltrate the credentials in `~/.claude`. [documented]
