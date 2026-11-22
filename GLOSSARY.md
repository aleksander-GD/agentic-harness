# Agent Harness

A personal setup for building software with AI coding agents. It also serves as evidence of the owner's engineering judgment.

## Language

**Field Notes**:
The owner's published technical notes on their portfolio. They explain the decisions behind the work and the reasoning for each.
_Avoid_: blog, dev log, journal

**Gate**:
A deterministic check whose result decides whether a step of work is done. Examples are a test suite, static analysis, and type checks. An agent's own claim that it has finished does not count.
_Avoid_: check, validation, verification step

**Behaviour List**:
The plain-language list of what a feature must do, stated in the domain's language with one behaviour per line and an ID for each. The agent drafts it, and it becomes the specification once the owner approves it.
_Avoid_: requirements, acceptance criteria, test plan

**Red Commit**:
The commit holding a feature's tests, written from the approved Plan and still failing. After it, the green step may not change them.
_Avoid_: test commit, spec commit

**Project Knowledge**:
The durable, owner-approved understanding of a project that every session starts from. It is the glossary, the ADRs and the Behaviour Lists, versioned with the code.
_Avoid_: memory, context, docs

**Task State**:
The short-lived record of where the current task stands and what is left. The agent writes it and it is thrown away when the task ends.
_Avoid_: progress notes, scratchpad, memory

## Runs

**Spec**:
A project's prioritised list of features for the harness to implement. A plain prompt counts as a Spec with a single feature.
_Avoid_: requirements document, PRD, backlog

**Run**:
One autonomous pass of the harness for one feature of a Spec, from Plan to pull request, stopping only where the owner must approve something.
_Avoid_: job, pipeline, session

**Plan**:
The agent's proposal for one feature: its Behaviour List and its approach. The owner approves it before any code is written, and it is the first of a Run's two stops.
_Avoid_: design doc, proposal, ticket

**Stage**:
One step of a Run, carried out by a fresh agent invocation or a Gate, in an order fixed by the run script.
_Avoid_: phase, step, agent

**Adoption**:
The one-time step that makes an existing repository ready for Runs. The harness maps the repository, proposes its Gates and a starting glossary, and the owner approves them.
_Avoid_: onboarding, import, setup

**Friction Log**:
The owner's running record of problems hit during Runs. Every addition to the harness must point to an entry in it.
_Avoid_: issues, backlog, retro notes
