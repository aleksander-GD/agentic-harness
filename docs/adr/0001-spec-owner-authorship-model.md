---
status: accepted
date: 2026-11-01
accepted: 2026-11-21
---

# The human owns the specification; the agent implements inside it

The harness exists mainly to produce evidence of its owner's engineering judgment, and only secondarily to produce code.

The owner therefore authors:

- the domain model and its language (the glossary);
- the Behaviour List, which states what the code must do;
- the decisions recorded in Field Notes.

Coding agents draft the test code from the Behaviour List and implement until the tests pass.

The tests stay independent of the implementing agent through four mechanisms. The fact that the human did the typing is not one of them:

1. **The Behaviour List is the owner's.** A missing behaviour is a gap in the owner's list, not an agent's oversight.
2. **Owner approval.** Every test is reviewed against the Behaviour List and the domain language, and the owner must be able to explain it.
3. **Locked tests.** Once approved, tests are committed as the red step. A Gate fails if the green step changes them, or changes the test-runner configuration.
4. **Mutation testing.** A Gate measures whether the tests catch deliberately injected bugs.

This deliberately departs from the "software factory" framing the project started with.

## Considered options

- **Factory.** Agents design, implement, test and review end to end; the human approves the merge request.
  - **For:** highest throughput.
  - **Against:**
    - an agent-written repository is weak evidence of its owner's skill;
    - tests that only the implementing agent has looked at are a weak check on that agent's code.
  - **Rejected.**
- **Spec-owner, with the human typing every test.**
  - **For:** the strongest independence.
  - **Against:** spends the owner's time on test boilerplate instead of on behaviour and design.
  - **Rejected** in favour of the variant chosen above.
- **Tutor.** The human writes all the code; the agent reviews and explains.
  - **Kept for:** the parts the owner most wants to showcase.

## Consequences

- The human bottleneck is now writing the Behaviour List and reviewing tests, not typing tests. This is by design.
- "The agent must not change the specification to get a pass" is enforced mechanically by the lock Gate, not by the prompt. The discovery log's experiment E2 measures how much difference that makes.
- A test that is genuinely wrong is changed by the owner, in a new red commit. The lock does not forbid test changes; it makes every test change a human decision.
- A red commit fails by design. So the full test suite must not run as a pre-commit hook. It runs before push, or on the pull request's latest commit.
- Most of the specialised agents originally proposed lose their reason to exist. See H3 in the discovery log.
