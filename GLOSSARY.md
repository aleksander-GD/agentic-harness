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
The owner's plain-language list of what a piece of code must do, written before any test or code exists. Each line is one expected behaviour, stated in the domain's language.
_Avoid_: requirements, acceptance criteria, test plan

**Red Commit**:
The commit holding approved, still-failing tests for one TDD cycle. Approving it fixes the specification that the green step must satisfy without changing it.
_Avoid_: test commit, spec commit

**Project Knowledge**:
The durable, owner-approved understanding of a project that every session starts from. It is the glossary, the ADRs and the Behaviour Lists, versioned with the code.
_Avoid_: memory, context, docs

**Task State**:
The short-lived record of where the current task stands and what is left. The agent writes it and it is thrown away when the task ends.
_Avoid_: progress notes, scratchpad, memory
