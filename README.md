# agentic-harness

A Claude Code plugin that turns a Spec into pull requests, one feature at a time. It runs on its own between two stops where the owner decides:

1. **Plan approval.** The owner approves the feature's Behaviour List and its approach.
2. **Pull request review.** The owner reviews the result.

Between those stops, deterministic Gates decide when the work is done: tests, static analysis, a lock on approved tests, and behaviour coverage.

> Status: v1 under construction. The design is settled; the code is being built slice by slice.

## Why it looks like this

The design came out of a recorded discovery process. Start here:

- [Discovery log](docs/discovery-log.md). It ends in the revised summary.
- [Architecture decision records](docs/adr/)
- [Glossary](GLOSSARY.md)

## Install

```sh
claude plugin marketplace add aleksander-GD/agentic-harness
claude plugin install harness@agentic-harness
```

### Local development

```sh
claude --plugin-dir ./plugin
```

Run `/reload-plugins` after editing.

## Layout

```text
.claude-plugin/marketplace.json   marketplace entry for the plugin
plugin/                           the harness plugin
docs/                             discovery log, ADRs, research, Friction Log, Run Log
```
