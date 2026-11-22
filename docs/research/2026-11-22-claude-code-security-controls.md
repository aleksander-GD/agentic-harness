# Claude Code security controls: what each one actually enforces

- **Date:** 2026-11-22
- **Sources:** the official docs at https://code.claude.com/docs/en/, specifically the sandboxing, permissions, settings, hooks, devcontainer and plugins/security pages, fetched on that date.
- **Status:** research input.

Each statement carries a tag:

- **[documented]**: the docs say it.
- **[inference]**: our own interpretation.

## Built-in sandbox

**What it covers.** It applies to **Bash only**. The docs state: "Claude's file tools, MCP servers, and hooks run outside it." [documented] It is off by default and is turned on with `sandbox.enabled`. [documented]

**OS mechanism.** It uses Seatbelt on macOS and bubblewrap on Linux/WSL2. Native Windows has no sandbox. [documented]

**Filesystem.**

- Writes are limited to the working directory, temp, and any added directories. [documented]
- Reads are allowed across the whole machine by default. [documented]
- Narrowing reads requires `sandbox.filesystem.denyRead` or `permissions.blockReadsOutsideWorkingDirectories`. [documented]

**Network.**

- A host-side proxy checks hostnames against `sandbox.network.allowedDomains`, which starts empty. [documented]
- The check is by domain only, and TLS is not inspected. The docs name domain fronting as a residual risk. [documented]

**Escape hatches.**

- `excludedCommands`.
- A retry with `dangerouslyDisableSandbox`. Setting `allowUnsandboxedCommands: false` turns this off.
- `!` shell-mode commands.

[documented]

**Gaps for the other tools.**

- `denyRead` does not stop the Read tool. [documented]
- `allowedDomains` does not limit WebFetch. [documented]
- WebFetch is therefore an outbound channel that the sandbox does not close. It has to be governed by permission rules. [inference]

## Permission rules

**Evaluation order.** Rules are checked deny, then ask, then allow, and the first match wins. An allow rule cannot override a deny. A deny at any scope beats an allow at any other scope. [documented]

**What `Read(./.env)` blocks.**

- It blocks the file tools. [documented]
- In Bash, it also blocks *recognised* commands such as `cat`, `head`, `tail`, `sed` and `tee`. [documented]
- It does **not** block `grep -r`, or a script that opens files itself. [documented]

**Not a boundary.** The docs state that a Bash deny or ask rule "isn't a security boundary around the program". They recommend combining permission rules with the sandbox for defence in depth. [documented]

## Hooks

- A PreToolUse hook that exits with code 2 blocks the call. A hook that times out does **not** block, unless it is configured with `onFailure: "block"`. [documented]
- Hooks run **outside** the sandbox, with full user permissions. Hooks that ship in a plugin are merged with the user's and the project's own hooks. [documented]
- So every hook, including those that ship in a plugin, is trusted code running at the owner's privilege level. Read it before installing it. [inference]

## Reference devcontainer

- Anthropic publishes a reference devcontainer, with `init-firewall.sh` restricting outbound traffic. It needs `NET_ADMIN` and `NET_RAW`. [documented]
- The docs describe it as "a working example rather than a maintained base image". [documented]

**Caveats the docs state.** [documented]

- When permission checks are skipped, a malicious project can exfiltrate whatever is inside the container. That includes the Claude credentials in `~/.claude`.
- Do not mount `~/.ssh` or cloud credentials into it.
- The agent can still modify the bind-mounted workspace on the host.

**The workspace is a channel back to the host.** [inference] Files the agent writes in the workspace can later run on the host, outside the container:

- `.git/hooks/*` scripts;
- `package.json` or Composer scripts;
- editor task files.

## Settings precedence

- Settings apply in this order: managed > command line > project local > shared project > user. [documented]
- A project's `permissions.allow` rules apply only after the owner trusts the workspace. `deny` and `ask` rules apply immediately. [documented]
- Not documented in the pages read: whether a plugin can ship `allow` rules that loosen restrictions set elsewhere.

## Implications for this project

These are all **[inference]**.

- Under threat model C14, the sandbox alone is not enough:
  - Read, WebFetch and MCP fall outside it.
  - Its default read access is machine-wide.
  - Its escape hatches must be turned off.
- Permission rules alone are not enough either. The docs themselves say Bash rules are not a boundary.
- A container puts *every* tool behind one OS boundary, including hooks, MCP and the file tools. It still leaves two exposures:
  - the Claude credential inside the container;
  - the workspace as a way back to the host.
