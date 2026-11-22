---
status: accepted
date: 2026-11-22
supersedes: ADR-0001
---

# Autonomous runs stop twice for the owner: at the Plan and at the pull request

The harness is now the primary evidence of the owner's engineering judgment. It runs on its own once it is given a Spec.

The owner keeps control of *what* gets built and *whether it ships*, at two stops:

1. **Plan approval.** For each feature, the agent drafts a Plan with two parts:
   - the Behaviour List, in the domain's language;
   - the implementation approach.

   The owner edits and approves it before any code exists.
2. **Pull request review.** Branch protection guarantees that nothing reaches `develop` without the owner's review.

Between those stops the agent works on its own:

- it writes the tests;
- it makes the Red Commit;
- it implements the feature;
- it answers to Gates, not to the owner:
  - tests;
  - static analysis;
  - the test lock;
  - a behaviour-coverage check that every approved behaviour has at least one tagged test.

## What carries over from ADR-0001

- The owner owns the specification, through the approved Behaviour List.
- A task is done when the Gates pass, not when the agent says so.
- Tests are locked after the Red Commit, so the agent cannot change the specification to get a pass.

## What changed, and why

- **The owner no longer writes the Behaviour List.** They approve it.
- **The owner no longer reviews tests before implementation.** They see the tests at pull request review instead.
- **The plan stop has no separate test list.** It would only duplicate the Behaviour List, so a deterministic coverage check replaces it.

The reason for these changes: a third stop for test review would defeat the purpose of an autonomous run.

## Considered options

- **Plan stop with the approach only, no behaviours.**
  - **Against:** the owner would no longer own what gets tested. Missing behaviours would only surface at pull request review, after the work is done.
  - **Rejected.**
- **Plan stop including a test list.**
  - **Against:** it repeats the Behaviour List.
  - **Rejected.**
- **A third stop to approve the Red Commit.** This was ADR-0001's model.
  - **For:** the strongest independence between tests and code.
  - **Against:** too many interruptions for an autonomous run.
  - **Rejected.**

## Consequences

- Nobody reviews the test bodies until the pull request, so weak tests are the main residual risk. Mutation testing is the planned remedy if the friction log shows weak tests getting through.
- Tests must be tagged with the IDs of the behaviours they cover, so the coverage check can be deterministic.
