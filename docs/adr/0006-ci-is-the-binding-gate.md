---
status: accepted
date: 2026-11-23
---

# CI is the binding Gate; gates inside a Run are a fast inner loop

A Claude Code workflow script cannot run commands or read files itself. Only the agents it starts can do that. So a gates Stage inside the `build` workflow would really be an agent saying "the tests passed", which is the agent's own claim, and the glossary's definition of a Gate excludes exactly that.

Therefore:

- **The binding Gate is CI on the pull request.** It runs the tests, static analysis, the test lock and behaviour coverage. Branch protection on `develop` requires those status checks to pass, so no agent's claim can merge code.
- **Inside a Run, the gates Stage is a fast inner loop.** It exists so that a Run can retry before it opens a pull request.
  - The Gate scripts write a result file holding their real exit code and output.
  - The workflow decides from that file, not from the agent's summary of it.
  - This makes misreporting harder, but it is still not proof. CI is the proof.

## Considered options

- **Trust the agent's report of the gate results.** Rejected: it is exactly the kind of claim a Gate exists to replace.
- **Run the gates only in CI, with no inner loop.**
  - **Against:** every failure would cost a full pull request round trip before the Run could retry.
  - **Rejected.**

## Consequences

- **Setup order.** The project template's setup script must create CI before it applies branch protection, because protection can only require status checks that already exist.
- **Reuse.** The Gate scripts must run the same way both locally and in CI, so one implementation serves both.
- **This amends ADR-0004**, which assumed the run script itself could execute Gates.
