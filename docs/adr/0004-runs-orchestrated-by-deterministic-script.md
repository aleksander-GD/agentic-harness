---
status: accepted
date: 2026-11-22
---

# A deterministic script orchestrates Runs, and state lives in files and git

A Run is driven by a run script written as ordinary code. The script decides which stage runs next. Each stage is a separate agent invocation with a fresh context, and the model never decides the order of stages.

Between stages, all state lives in files and git:

- the Plan;
- the Red Commit;
- the Gate results;
- the review notes.

At a stop, the script writes what the owner needs to see and then exits. After the owner approves, a resume command continues the Run.

**Why the script orders the stages, not a model.** The order of stages is already known (plan, red, green, Gates, review, pull request). Letting a model choose the order adds cost and unpredictability without adding information. A script can also be tested.

**Why each stage gets a fresh context.** Context rot is documented. A stage that starts fresh reads only the files the previous stage left behind, so nothing stale carries over in the context.

**Why state lives in files and git.** A crash, a closed laptop or a long pause for review loses nothing. Every handover between stages can also be inspected.

## Considered options

- **An LLM orchestrator agent that delegates to sub-agents.** Rejected: it adds non-determinism to a fixed sequence, and its own context grows across stages.
- **One long agent session with pauses.** Rejected: a long-lived process has to wait for the owner, and its context accumulates every stage's leftovers.

## Consequences

- The run script is the main piece of harness code, so it is built test-first like any other code.
- The interactive main session (C11) is still the way to work outside Runs.
- v1 implements the run script as Claude Code workflows inside the plugin, not as a separate command-line tool. See [ADR-0005](0005-v1-runs-as-plugin-workflows-on-subscription.md).
