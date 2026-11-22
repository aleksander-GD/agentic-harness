# Discovery log

This is a living record of the architecture discovery, kept in chronological order. Every entry is labelled as one of:

- **Confirmed**: the owner stated it or agreed to it.
- **Hypothesis**: believed, but not yet tested.
- **Open**: not yet answered.
- **Experiment**: a test that would settle something.

Speculation is never recorded as Confirmed.

Research input: [reference review](research/2026-10-30-reference-review.md)

---

## Round 1 (2026-10-30): what is this for?

**Question asked.** Which concrete, repeatable task is this for, and where does a single plain Claude Code session fail at it today?

**Answer, paraphrased.**

- **What it's for:** building software, mainly personal projects, plus occasional small tasks on client projects.
- **How I want to work:** like a backend developer, using TDD and focusing code quality on domain logic.
- **Why:** to understand every technical decision well enough to show employers I am still hands-on technical.
- **How that gets shown:** the evidence is published as [Field Notes](../GLOSSARY.md) on the portfolio.
- **What I don't want:** to be seen as a project manager.

**What the answer did *not* contain:**

- a concrete task;
- an observed failure of a plain Claude Code session.

So no productivity problem has been demonstrated yet. Category (f), "haven't hit a wall", applies.

### Confirmed

- **C1. Purpose.** The primary purpose is career repositioning, from project manager to hands-on software engineer. The secondary purpose is learning how agents work. Being productive is a means, not the goal.
- **C2. Way of working.** The owner wants to keep working test-first, with domain logic at the centre of design and quality.
- **C3. Where evidence goes.** Evidence of judgment is published through Field Notes.
- **C4. Initial domain.** The first domain is software development: personal projects first, client tasks occasionally.

### Tension surfaced

**T1. "Software factory" versus "prove I'm technical".** A factory implies the agent produces the code. The career goal requires visible human authorship of the design, the tests and the decisions.

The more autonomous the system, the weaker the evidence it produces about its owner. The architecture has to settle who authors what. See Round 2.

### Hypotheses

- **H1.** Most of the agents in the original table, such as Orchestrator, File Reader, Lead and Deployer, are not needed to serve C1–C4. *How to test:* the Round 2 decisions, then the baseline experiment E1.
- **H2.** The harness itself is a weaker portfolio signal than projects built with it plus Field Notes that explain the decisions behind them. *How to test:* Q3 in Round 2.

### Experiments

- **E1. Baseline.** Run 5–10 real tasks on the first project with plain Claude Code and no harness. Log time, tokens or cost, and how each task failed, using categories (a)–(f).
  - *Why:* every later piece of the harness has to justify itself against this log.
  - *Blocked on:* choosing the first project.

### Constraints noted

- **K1.** Client work is subject to the employer's AI policy. That rules out production access and the use of non-public personal data. This limits what the harness may touch on client projects, and the security model has to treat it as a hard boundary.

---

## Round 2 (2026-11-01): authorship, showcase, scope

### Confirmed

- **C5. Authorship model: spec-owner.** The owner owns the domain model and the tests. The agent implements inside those constraints, and deterministic gates decide when work is done. See [ADR-0001](adr/0001-spec-owner-authorship-model.md). The owner asked for more detail before this is final, so the ADR stays *proposed* for now.
- **C6. What the showcase is.** The showcase is the owner's knowledge and what they learn along the way. It is made of the projects built, the Field Notes, and the harness. The harness is one *part* of the showcase, not the whole of it.
- **C7. Projects.** The candidate projects each get their own discovery session and are out of scope here:
  - a sports-shooting score tracker;
  - a simpler remake of an existing app;
  - the owner's portfolio site.
- **C8. Prior work.** An earlier repository, `agentic-scaffold`, was started as a starter template meant to work with any agent runtime. The idea was that each new project starts with guardrails and rules already in place. It has not been reviewed in this discovery yet.

### Tension surfaced

**T2. "An agent framework I can orchestrate" (owner, Q3).** This wording pulls back towards an LLM orchestrator. Under C5, most coordination is already done by the owner plus a fixed sequence of steps. "Orchestrate" needs a precise meaning: see Round 3, Q5.

### Hypotheses

- **H3. Mapping the original agents onto C5.** Proposed:

  | Original agent | Proposed replacement |
  |---|---|
  | Builder | The single main agent, during the green step |
  | Tester | The owner, who writes tests, plus a deterministic test runner |
  | Lead | The owner |
  | Reviewer | A sub-agent with fresh context; justified only by its independence |
  | Security Auditor | Deterministic scanners |
  | Deployer | CI |
  | PR Creator | A command |
  | Orchestrator | A defined workflow that the owner triggers |
  | Learner | The Field Notes workflow |

  Not yet validated.
- **H4. Portability comes from layering.** Runtime-agnostic guardrails can come from *where* they sit, without a runtime abstraction layer. Tests, static analysis, git hooks, CI, container isolation and the absence of credentials all live below the agent runtime, so they are portable for free. Only a thin layer is specific to Claude Code: skills, tool hooks, and permission settings. *How to test:* review `agentic-scaffold` against this split.
- **H5. Agents will try to game the tests.** In green steps, agents will sometimes edit tests or special-case them to get a pass. If so, this has to be enforced mechanically, not by instruction. *How to test:* E2.

### Experiments

- **E2. Gaming the tests.** During E1, count how often the agent edits or weakens tests, or hard-codes results, when told only in the prompt not to.
  - Repeat with a hook that blocks test edits.
  - Then run a mutation-testing tool to measure how strong the tests themselves are.

---

## Round 3 (2026-11-01): orchestration, portability, test authorship

Research input: a review of the employer's internal `agent-config` repository. Under K2, that review is kept privately and is not part of this repository.

### Answers, paraphrased

- **Q5 (orchestration).** The owner leaned towards an orchestrator agent, but one that "follows the steps" of a defined workflow and "knows what to use when".
- **Q6 (portability).** The harness should be a *configuration* (skills, hooks, rules) kept in its own repository and loaded into projects, modelled on the employer's `agent-config`.
- **Q7 (tests).** The agent drafts all tests. The owner reviews them, approves or rejects each one, and must be able to explain all of them.
- **Q8 (first project).** Out of scope for this discovery. Projects are decided in their own sessions.

### Confirmed

- **C9.** The harness configuration lives in its own repository, separate from the projects it is used in.
- **C10.** Project choice and project design are outside this discovery's scope.

### Tensions surfaced

- **T3. Orchestrator agent vs. workflow.**
  - The behaviour the owner described (a fixed sequence, tools chosen per step) is what a *single main session* does when it has a workflow skill loaded. Skill descriptions are already the mechanism for "knowing what to use when".
  - `agent-config` shows that an autonomous multi-step flow can run with no orchestrator agent at all.
  - A separate orchestrator agent is only justified when the coordinator's context must stay clean while workers use up tokens, for example in long multi-ticket runs. Nothing observed so far needs that.
  - To be resolved in Round 4.
- **T4. Agent-written tests vs. ADR-0001's rationale.**
  - ADR-0001 relies on the tests being independent of the implementing agent. If the agent writes the tests and the owner only reviews them, that independence rests entirely on the review.
  - Reviews catch bad tests that are *present*, such as tests that check a variable name. They are much worse at noticing tests that are *absent*.
  - To be resolved in Round 4.
- **T5. A template, or loaded configuration?**
  - `agentic-scaffold` (a template copied into each project) and `agent-config` (configuration loaded from a shared repository) are two different distribution models.
  - Each fits a different layer from H4:
    - The bottom layer (CI, test and static-analysis configuration, git hooks) has to live *inside* each project, because CI runs it there.
    - The runtime layer (skills, tool hooks) can be loaded from elsewhere.
  - To be resolved in Round 4.

### Constraints noted

- **K2.** Content from the employer's internal repositories, such as `agent-config`, may inform the design. It must not be copied into the public portfolio or harness repositories.

---

## Round 4 (2026-11-21): orchestration settled, distribution, test independence

### Answers, paraphrased

- **Q9.** Agreed.
- **Q10.** Agreed.
- **Q11.** Agreed. The owner added that the Behaviour List carries their domain knowledge and expectations, and that tests should be checked against it.
  - The owner did not yet follow the "lock" and "mutation testing" points. These are explained in the conversation for this round.

### Confirmed

- **C11 (resolves T3). The main session is the orchestrator.**
  - A workflow skill defines the steps.
  - Gates enforce the transitions between steps.
  - Within each step, the model chooses which skills, tools and sub-agents to use.
  - A separate orchestrator agent is deferred until evidence shows it is needed, such as a coordinator whose context must be protected during long runs over many tickets.
- **C12 (resolves T5). Two artifacts.** A project template carries the portable layer, and a configuration plugin carries the runtime layer. See [ADR-0002](adr/0002-project-template-plus-config-plugin.md).
- **C13 (resolves T4). The agent drafts the tests, and their independence comes from four mechanisms:**
  - the owner writes the Behaviour List;
  - the owner approves each test against that list and the glossary;
  - Red Commits lock the tests and the test-runner configuration;
  - mutation testing measures how strong the tests are.

  [ADR-0001](adr/0001-spec-owner-authorship-model.md) is now *accepted*. This supersedes the "stays proposed" note in C5.

### Hypotheses revised

- **H3.** The "Tester" row now reads: the agent drafts the tests from the owner's Behaviour List, the owner approves them, and Gates (the lock and mutation score) back them up.
  - The "Orchestrator" row is confirmed by C11.
  - The other rows are still unvalidated.

### Known friction

- A Red Commit fails by design. It conflicts with:
  - any pre-commit hook that runs the test suite;
  - CI that runs on every push.

  The suite therefore runs before push, or on the pull request's latest commit, and never at pre-commit. This is recorded in ADR-0001.

---

## Round 5 (2026-11-22): security, threat model and scope

### Confirmed

- **C14. Threat model: both accidents and manipulation, with prompt injection driving the design.**
  - **Accidents:** a well-meaning agent runs a destructive command or reads secrets while debugging.
  - **Manipulation:** instructions planted in content the agent reads, such as dependencies, issues, web pages and code comments.
  - **Consequence:** denylist-style guardrails such as pattern-matching hooks only count as protection against accidents. Every control meant to stop manipulation must still hold when the agent is actively looking for a way around it.
- **C15. Client projects are out of the MVP.**
  - The MVP covers personal projects only.
  - Client work enters later as a deliberate decision with its own ADR, once the security model has been proven on the owner's own code (see K1).

### Framing adopted for the security design

The "lethal trifecta" describes when an agent can be turned against its owner. It needs all three of:

1. access to private data;
2. exposure to untrusted content;
3. a way to communicate externally.

A coding agent cannot avoid leg 2. The design therefore removes legs 1 and 3:

- **Leg 1:** keep valuable credentials out of the agent's environment (Q15).
- **Leg 3:** restrict outbound network access (Q14).

### Open

- **Q14. Where the agent runs.** The options are the host machine with permissions only, Claude Code's built-in sandbox, or a container with an outbound-network allowlist. This is waiting on a check of Claude Code's documented sandbox capabilities.

Research input for Q14: [Claude Code security controls](research/2026-11-22-claude-code-security-controls.md).

### Experiments

- **E3. Red-team the harness itself.**
  - Plant an injection in a fixture file of a test project, such as a README or a code comment.
  - The injection tells the agent to read a fake key and send it to a fake external host.
  - Run it with each isolation configuration.
  - For every layer, record whether it blocked the attempt, what the attempt was, and where it was blocked.
  - Use fake credentials only.
  - *Why:* this turns "the security model works" from a claim into evidence. It is also strong material for Field Notes.

---

## Round 6 (2026-11-22): isolation, credentials, git safety

### Confirmed

- **C16 (resolves Q14). Isolation scales with autonomy.**
  - **Interactive sessions:** the built-in sandbox, configured strictly, plus permission rules. The owner watching the session is part of the boundary.
  - **Unattended runs:** a container with an outbound firewall and no credentials mounted. This is mandatory, and it is deferred until unattended runs are wanted.
  - The MVP is interactive only.
- **C17 (Q16). Branch protection on `main` and `develop`.**
  - Remote branch protection is the boundary.
  - Local "ask" rules for destructive git commands guard against accidents.
  - The owner will set this up "when I get to that". That leaves a risk that it is forgotten (see Round 7).

### Challenged

- **Q15. "No valuable credentials where the agent runs."**
  - **Owner's point:** integrating an external system needs a real API key in the local `.env`.
  - **Current state:** no real keys are held locally today.
  - **Consequence:** this is a valid limit on the principle as first written. If the agent can run code that loads a key, it can obtain that key. Denying it read access to `.env` does not change that.
  - **To refine in Round 7:**
    - prefer provider test keys, or scoped keys with a spending cap;
    - tests the agent runs use fakes, and never real keys;
    - real calls are made only by the owner or by CI;
    - the outbound-network allowlist is the backstop if a key leaks.

---

## Round 7 (2026-11-22): credentials refined, enforcing setup, memory

### Confirmed

- **C18 (resolves Q15/Q18). Credentials for external integrations.**
  - **Keys:** use a provider test key where one exists. Otherwise use a dev-only key that is scoped to the minimum and has a spending cap.
  - **Agent tests:** they use fakes. `.env.testing` holds no real key.
  - **Real calls:** made by the owner, or by a smoke test running in CI with a CI secret.
  - **Backstop:** the sandbox's outbound allowlist limits where a leaked key could be sent.
  - **Principle:** any credential the agent can reach must be low-value. It must also be absent from the code paths the agent runs.
- **C19 (Q19). Branch protection is applied by the project template's setup script, not done by hand.** Open fact to check: whether the owner's GitHub plan supports protection on private repositories.
- **C20 (Q20). Persistent project memory is curated, not generated.**
  - The memory consists of the glossary, the ADRs and the Behaviour Lists. They are versioned with the code and reviewed in pull requests.
  - The agent reads the code fresh each session.
  - The MVP has no generated index, code map or summary store.
  - This should be revisited only if E1 shows that re-exploring the code costs significant tokens.
  - The owner supplied best-practice sources to check this against.

Research input: [context and memory sources](research/2026-11-22-context-memory-sources.md). C20 holds after checking it against the owner's sources.

### Hypotheses

- **H6. Keep CLAUDE.md short for quality, not cost.**
  - Stable instructions are prompt-cached: reads cost 0.05–0.1× the normal input price. So trimming them saves little money.
  - Every line still takes space in the context window, and context rot is documented.
  - The real per-session cost is probably fresh exploration of the code, which caching does not cover. E1 should measure this separately.

### Tension surfaced

- **T6. Project knowledge vs. task state.**
  - The owner's sources recommend agent-written progress files.
  - C20 covers *curated* project knowledge.
  - These are different things, with different authors and different lifetimes. See Round 8.

---

## Round 8 (2026-11-22): memory edits and payment model

### Confirmed

- **C21 (resolves T6, Q21). The agent proposes, the owner approves.**
  - The agent may *propose* edits to Project Knowledge in the same diff as its code change, and the owner approves or rejects them. It never writes them on its own authority. This is the same pattern as tests (C13).
  - Task State is written freely by the agent and thrown away when the task ends.
  - The tool's own memory, which it keeps outside the repository, may hold personal preferences, but never Project Knowledge.
- **C22 (Q22). The owner pays by subscription, not per token.**
  - "Cost" therefore means *usage limits*: how much work fits into a day or a week. It does not mean dollars.
  - Optimising tokens matters only if those limits are actually hit. E1 should record usage against the limits.

---

## Round 9 (2026-11-22): models, Learner, metrics

### Confirmed

- **C23 (Q23). No model routing in the MVP.**
  - **Main session:** one model. Effort level is the main thing to tune.
  - **Read-only search sub-agents:** a smaller model is allowed here, because a miss is cheap and gets re-checked.
  - **Never a smaller model for:** implementing, reviewing, or approving.
  - **Revisit:** only if E1 shows the usage limits are being hit.
  - **Rejects:** the owner's original hypothesis that smaller models should read the codebase to save tokens.
  - **Evidence:**
    - Spotify's own report: the cheap model missed a bug, and edits could not be delegated.
    - Claude docs: "tuning effort is often a better lever than switching models".
    - The subscription model (C22).
- **C24 (Q24). The Learner is a skill, not an agent.**
  - It runs on demand. It explains, compares alternatives, and quizzes the owner.
  - It may produce outlines for Field Notes. The owner writes every Field Note in their own words.
  - **Owner's addition:** the Learner must cover *every test* and *every significant decision*.
- **C25 (Q25). Four metrics, recorded per task by hand:**
  - **Rework:** how often the owner corrected or redirected the agent.
  - **Escaped defects:** bugs found after a task was considered done.
  - **Spec tampering:** attempts to change tests during the green step.
  - **Usage:** share of the subscription limit consumed.

  Recorded for 5–10 tasks before the harness (E1) and 5–10 after.

  Deliberately excluded:
  - time saved, because it cannot be measured honestly;
  - lines of code, because they are meaningless;
  - mutation score, because it measures the tests, not the harness.

### Hypotheses revised

- **H3.** "Learner → the Field Notes workflow" is refined by C24: a teaching skill, with the owner as author.

---

## Round 10 (2026-11-22): MVP scope and where the docs live

### Confirmed

- **C26 (Q26). MVP scope and order.**
  1. **Safety basics.**
     - Sandbox on, with WebFetch and destructive git commands set to "ask".
     - No real secrets on the machine.
     - Branch protection.
  2. **Baseline (E1).** The first 5 tasks of the first project, done with plain Claude Code. Record the four metrics (C25) for each.
  3. **Build v0.**
     - **Project template:** CI running the tests and static analysis; the test-lock Gate; a setup script that creates `develop` and applies branch protection.
     - **Configuration plugin:** the TDD workflow skill, the Learner skill, and one review sub-agent with fresh context.
  4. **Measure again.** The next 5 tasks with the harness, recording the same four metrics.
  5. **Write 3–4 Field Notes.**
  - **Time box:** about 6 weeks.
- **C27 (Q27). Where things live.**
  - **This repository** (`agentic-harness`) is the harness. It holds the glossary, the ADRs, the discovery log and the research notes.
  - **The `agent-config` review** stays outside it (K2).
  - **`agentic-scaffold`** is abandoned and will not be reused.

---

## Discovery summary (2026-11-22)

> **Superseded** by the [revised summary](#discovery-summary-revised-2026-11-22) at the end of this log. Discovery was reopened in Round 11.

### Problem statement

The owner is moving from project management back to hands-on software engineering. They need a way of building software with AI agents that:

- keeps them the author of the domain model, the specification and the decisions;
- produces publishable evidence of their engineering judgment (Field Notes);
- teaches them how agents, context and security actually work.

Being productive matters, but it is secondary. The discovery found no productivity problem yet, so the baseline (E1) has to establish one.

### Architectural direction

**One agent and one workflow, plus deterministic Gates.** The harness is shipped as two artifacts: a project template and a configuration plugin.

| Concern | Decision | Ref |
|---|---|---|
| Who authors what | Owner: domain model, Behaviour List, approval of tests, Field Notes. Agent: drafts tests, implements. | C5, C13, ADR-0001 |
| Orchestration | The main session, driven by a workflow skill. Gates enforce the transitions. No orchestrator agent. | C11 |
| Specialised agents | Only a review sub-agent with fresh context. Every other role is the owner, a skill, a Gate or CI. | H3, C24 |
| Distribution | Project template for the portable layer; Claude Code plugin for the runtime layer. | C12, ADR-0002 |
| Test independence | Behaviour List, owner approval, locked Red Commits, and (later) mutation testing. | C13 |
| Memory | Curated Project Knowledge in the repository. Task State is temporary. The agent proposes changes and the owner approves them. No generated index. | C20, C21 |
| Models | One model. Effort level is the dial. A smaller model only for read-only search. | C23 |
| Security | Threat model covers accidents and injection. Isolation scales with autonomy. Low-value credentials only. Branch protection applied by script. | C14–C19 |
| Learner | A teaching skill that explains every test and every decision. The owner writes the Field Notes. | C24 |
| Measurement | Rework, escaped defects, spec tampering and usage, recorded before and after. | C25 |

### Rejected

- **Factory model.** Weak evidence of the owner's skill, and weak independence of the tests.
- **11-agent hierarchy.** Most roles collapsed into the owner, skills, Gates or CI.
- **Model routing for reading.** Fidelity loss, and no cost benefit under a subscription.
- **Generated codebase memory.** It goes out of date without anyone seeing it, and context rot is documented.
- **Abstraction over agent runtimes.** Portability comes from layering instead.
- **User-scope symlink install.** Different projects need different Gates.
- **Hooks as the boundary for secrets.** They work only against accidents.

### Deferred, each with the trigger that brings it back

| Item | Trigger |
|---|---|
| Orchestrator agent | Long unattended runs over many tickets, where the coordinator's context must be protected |
| Model routing | E1 shows usage limits being hit |
| Generated index or code map | E1 shows that re-exploring the code costs a lot |
| Devcontainer with firewall | The first unattended run |
| Mutation testing | After the first features exist |
| E3 red-team | After the MVP works |
| Client projects | Security model proven, plus its own ADR |
| Research and PM domains | A concrete need, with its own discovery |

### Open facts to check during implementation

- Can a Claude Code plugin ship sandbox and permission settings? If not, they go into the template's `.claude/settings.json`.
- Does the owner's GitHub plan allow branch protection on private repositories?

### Hypotheses not yet validated

- **H3:** the remaining rows of the agent mapping.
- **H4:** portability through layering.
- **H5:** agents game tests unless the tests are locked.
- **H6:** the session cost lies in exploring code, not in instructions.

### Experiments

- **E1. Baseline.** Blocked until the first project is chosen in its own session.
- **E2. Spec tampering**, measured with and without the lock.
- **E3. Red-team the harness itself.** Post-MVP.

---

## Closure (2026-11-22)

The owner confirmed a shared understanding of the design. Discovery is closed.

**Next steps:**

- Run a separate discovery for the first project. This unblocks E1.
- Then follow the MVP order in C26.

---

## Round 11 (2026-11-22): discovery reopened, the harness becomes the primary evidence

### What changed

**The owner's clarified goal.** The harness is the primary evidence. It is an environment that runs on its own once it is given a feature specification, and it stops only at stages that need the owner's judgment. At minimum those are the plan and the final review.

Secondary evidence:

- what is built with the harness;
- how the harness is improved over time.

Field Notes sit alongside this; they are not the centre of the portfolio.

**Decisions this supersedes or reopens:**

- **C6 (what the showcase is).** Superseded: the harness is now the primary evidence, not one part of it.
- **C26 (MVP scope and order).** Reopened: the MVP is now the autonomous loop itself. Building first and measuring a baseline first are no longer in order.
- **C11 (orchestration).** Refined: autonomous runs are driven by a deterministic run script. The model runs inside each stage, never between stages.
- **C16 (isolation scales with autonomy).** Unchanged, but its consequence moves forward: the devcontainer is now part of the MVP, because autonomous runs are unattended by definition.
- **ADR-0001 (spec-owner authorship).** Under review: which stages stop for the owner is still open (Q28).

### Confirmed

- **C28 (Q29). How a run works.**
  - **Starting a run.** The owner starts a run with one command, which takes either a Spec or a plain prompt. A prompt is the usual input for a single feature, or for work on an existing repository.
  - **Isolation.** The run executes headless inside the devcontainer.
  - **Stages.** Each stage is a separate agent invocation with a fresh context. The run script decides the order of stages.
  - **State.** State between stages lives in files and git; nothing waits in memory. At a stop, the script writes its output and exits, and a resume command continues the run after the owner approves.
  - **Output.** A pull request into `develop`. That pull request is the final review, and branch protection enforces it.

### Open

- **Q28. Which stages stop for the owner.** The owner is unsure about option (b), which combines a plan with a test list.
- **Q30. How a multi-feature Spec becomes Runs.** Either one plan stop per feature, or planning several features up front.

---

## Round 12 (2026-11-22): the Plan stop and multi-feature Specs

### Confirmed

- **C29 (Q28). The Plan stop holds behaviours and approach only.**
  - The agent drafts the Behaviour List and the approach. The owner edits and approves both.
  - There is no separate test list, because it would duplicate the Behaviour List (the owner's objection).
  - A deterministic Gate checks that every approved behaviour has at least one test tagged with its ID.
  - This changes authorship. The owner *approves* the Behaviour List instead of writing it, and first sees the tests at pull request review.
  - See [ADR-0003](adr/0003-autonomous-runs-with-two-owner-stops.md), which supersedes ADR-0001.
- **C30 (Q30). One feature at a time.** For each feature the Run goes:
  1. Plan, then the owner's approval;
  2. autonomous implementation;
  3. pull request, then the next feature.

  Plans are always written against current code. Pipelining (planning the next feature while the owner reviews the current pull request) is added only if the friction log shows the stops becoming a bottleneck.
- **C31.** Runs are orchestrated by a deterministic script, and state between stages lives in files and git. See [ADR-0004](adr/0004-runs-orchestrated-by-deterministic-script.md). This records the reasoning behind C28.

### Risks

- **R1. Weak tests.** Nobody reviews test bodies before the pull request, so tests that pass but prove little are the main residual risk. Mutation testing is the candidate fix. It enters only when the friction log shows weak tests getting through.

---

## Round 13 (2026-11-22): the v1 Run, how it is invoked, existing repos

### Confirmed

- **C32 (Q31). The v1 Run for one feature has six stages:**
  1. **Plan.** Ends at a stop for the owner's approval.
  2. **Red.** Tagged tests, committed as the Red Commit, which must fail.
  3. **Green.** The implementation.
  4. **Gates:**
     - tests;
     - static analysis;
     - the test lock;
     - behaviour coverage.
  5. **Review.** A fresh agent reviews the diff against the Plan and the glossary.
  6. **Pull request.** Ends at a stop for the owner's review.

  **Retries.** Gate failures and serious review findings both send the Run back to Green. They share a budget of 3 attempts. After that, the Run stops and reports what is still failing. It never loops without a limit.
- **C33 (Q32). Stages are invoked headless with `claude -p`, from a run script written in TypeScript and built test-first.**
  - **Facts to check before building:**
    - Do headless runs authenticate through the owner's subscription?
    - Do the subscription terms allow automated runs?
  - **New input from the owner.** A colleague's harness lives entirely inside a Claude Code plugin: flows, agents, hooks and guardrails. It shows, in a sidebar inside Claude Code, which agents are working and which have stopped to wait for a human.
  - **Consequence.** Visibility of a Run's state is a new requirement. It also reopens *where* the run script lives: outside Claude Code as a command-line tool, or inside it as a plugin. See T7.

### Tension surfaced

- **T7. An external command-line tool or an in-app plugin?**
  - **External tool calling `claude -p`:**
    - **For:** headless; works inside the devcontainer; easy to test.
    - **Against:** no visualization unless it is built.
  - **Plugin running inside Claude Code:**
    - **For:** the app's own status display and "needs attention" signals come for free.
    - **Against:** it may not run unattended inside the devcontainer, which C16 requires. Pauses may also not survive the way ADR-0004 requires.
  - **Status:** waiting on a check of Claude Code's documented capabilities.

### Challenged

- **Q33. Existing repositories.**
  - **The owner wants them supported**, with an explorer agent that gathers the information needed.
  - **Explorer agent:** acceptable as a *read-only search sub-agent*, which C23 already allows. Context isolation is the strongest documented reason for having a sub-agent.
  - **The real problem with existing repositories is Gates, not information.** A repository without tests or static analysis gives a Run nothing deterministic to answer to. An explorer can describe the repository, but it cannot make the repository checkable.
  - **Scope:** open (Round 14).

### T7 update: findings from the docs check

Source: [runtime options](research/2026-11-22-runtime-options.md).

- **The terms are the deciding fact.** The Consumer Terms bar scripted, non-human access except through an API key or with explicit permission. The docs do support subscription tokens for CI, so whether unattended `claude -p` on a subscription is allowed remains unresolved.
- **This puts C33 at risk.** A headless command-line tool running on the subscription (C22) rests on a question about the terms.
- **Claude Code workflows fit the design.** A workflow is a deterministic JavaScript orchestrator, shippable in a plugin and watchable with `/workflows`. Workflows cannot pause for a human, but a Run's two stops fall exactly on workflow boundaries: a *plan* workflow, then a *build* workflow. State between the two lives in files and git, which keeps ADR-0004's intent.
- **Visibility is native.**
  - Agent view shows Working, Needs input and Completed.
  - The `Notification` hook and desktop notifications cover "needs attention".
  - Mods can draw a sidebar pane, as the colleague's harness does.
- **Open decision (Round 14):** whether Runs are paid for by subscription, which means started by the owner inside Claude Code, or by API key, which allows fully unattended runs.

---

## Round 14 (2026-11-22): visibility, existing repos, time box

### Confirmed

- **C34 (Q34). Visibility is a v1 requirement.** The owner can see each Run's current Stage and is notified when a Run needs them. The mechanism is decided with T7.
- **C35 (Q35, partly). Adopting existing repositories.**
  - Existing repositories are supported through a one-time *adopt* step:
    1. A read-only explorer sub-agent maps the repository.
    2. It proposes a Gate configuration and a starting glossary.
    3. The owner approves them, once per repository.
  - Runs are refused until the repository has a runnable test command.
- **C36 (Q36). No time box.** The owner considers it irrelevant. This removes the 6-week time box from C26.

### Challenged

- **Q35. The owner wants client repositories supported as well as personal ones.** This reverses C15, which kept client projects out of the MVP, and runs into K1, the employer's AI policy. Round 15 resolves it.

---

## Round 15 (2026-11-22): client repos, runtime

### Confirmed

- **C38 (Q38). Client repositories stay out of v1. C15 stands.**
  - Adoption (C35) applies only to personal repositories for now.
  - According to the owner, the employer has no say in this.
  - The remaining conditions are technical, and client repositories come back as their own decision once all of them are handled:
    - credentials are low-value only (C18);
    - the harness never reads real personal data;
    - nothing touches production.

- **C37 (Q37). v1 runs on the subscription, and the owner starts each Run inside Claude Code.**
  - **How a Run works:** two plugin workflows, `plan` then `build`. The `build` workflow runs in the devcontainer.
  - **Visibility:** native to Claude Code.
  - **What changes:** this supersedes C33 and resolves T7. See [ADR-0005](adr/0005-v1-runs-as-plugin-workflows-on-subscription.md).
  - **Deferred:** unattended Runs paid for with an API key are the upgrade path.

---

## Round 16 (2026-11-22): v1 scope and evidence

### Confirmed

- **C39 (Q39). v1 scope.** This supersedes C26.
  - **The plugin:**
    - `plan`, `build` and `adopt` workflows;
    - `explorer` and `reviewer` agents;
    - skills for TDD conventions and the Learner;
    - hooks for the test lock and for notifications.
  - **The project template:**
    - CI running the Gates;
    - a devcontainer;
    - a setup script that creates `develop` and applies branch protection;
    - `.claude/` settings, as a fallback.
  - **A side pane** is optional. Agent view covers C34.
- **C40 (Q40). Evidence comes from the four metrics (C25), recorded for every Run, plus a Friction Log.**
  - Every addition after v1 must cite a Friction Log entry, and its effect is measured across Runs.
  - E1, the baseline without the harness, is retired. A claim that "the harness beats plain Claude Code" is therefore out of scope.

**The owner asked why** the template holds CI and a devcontainer, and why model routing is excluded. The answers are already in ADR-0002 and C23. They were re-explained in conversation, and the decisions stand.

---

## Discovery summary (revised, 2026-11-22)

### Problem statement

The owner is moving from project management back into hands-on software engineering.

**The harness is the primary evidence.** It is an environment that takes a Spec or a prompt and implements it feature by feature, on its own. It stops only for:

- the owner's approval of each Plan;
- the owner's review of each pull request.

**The secondary evidence** is what gets built with the harness, and how the harness is improved, guided by the Friction Log and the metrics. Field Notes sit alongside both.

### Architecture

| Concern | Decision | Reference |
|---|---|---|
| Owner's stops | Plan approval (behaviours and approach), then pull request review | C29, ADR-0003 |
| Orchestration | Deterministic Claude Code workflows (`plan`, `build`, `adopt`). Every Stage runs in a fresh context, and state lives in files and git | C31, ADR-0004, ADR-0005 |
| Runtime | Subscription. The owner starts each Run in Claude Code, and `build` runs in the devcontainer | C37, ADR-0005 |
| Distribution | The plugin holds how agents work. The project template holds what each repository enforces | C12, ADR-0002 |
| Agents | Explorer (read-only) and reviewer (fresh context). Every other role is a workflow, a skill, a Gate or CI | C39 |
| Done | Gates decide: tests, static analysis, the test lock, behaviour coverage. Retries share a budget of 3, then the Run stops and reports | C32 |
| Memory | Curated Project Knowledge in the repository. The agent proposes changes and the owner approves them | C20, C21 |
| Models | One model for the Run. A smaller model is allowed for the explorer only. No routing | C23 |
| Security | Threat model covers both accidents and injection. Isolation scales with autonomy. Credentials are low-value only. Branch protection is applied by script | C14–C19 |
| Repositories | Template repositories, plus personal repositories through `adopt`. No client repositories | C35, C38 |
| Visibility | Agent view, `/workflows`, and notification hooks | C34 |
| Evidence | The four metrics for every Run, plus the Friction Log | C25, C40 |

### Deferred, with the trigger that brings each back

| Item | Trigger |
|---|---|
| Unattended Runs paid for with an API key | Runs should start without the owner |
| Model routing | Usage limits are hit, or Runs move to an API key |
| Mutation testing | The Friction Log shows weak tests getting through (R1) |
| Planning the next feature while the owner reviews the current one | The owner's stops become the bottleneck |
| Generated index or code map | The Friction Log shows re-exploring the code is costly |
| Side pane | Agent view is not enough |
| Client repositories | All technical conditions in C38 are met |
| Research and PM domains | A concrete need appears |

### Open facts to check while building

- Can a plugin ship sandbox and permission settings?
- Plugin sub-agents ignore `permissionMode` and `hooks`. Their guardrails must therefore come from settings or the sandbox.
- Can agent view and notifications see a CLI session running inside the devcontainer?
- Does the GitHub plan support branch protection on private repositories?

### Experiments

- **E2. Spec tampering**, with and without the test lock.
- **E3. Red-team the harness**, after v1.

---

## Closure, revised (2026-11-22)

The owner confirmed a shared understanding of the revised design "for now". Discovery is closed again.

**Next step:** build v1 as scoped in C39.

If anything changes during the build, it enters through the Friction Log and new rounds or ADRs. It is never rewritten silently.

---

## Build notes (2026-11-23)

These are decisions taken while starting to build v1. Each one either fills in a detail that discovery left open, or was forced by a fact found in the docs.

### Confirmed

- **C41. Supported stacks.** The template supports TypeScript (Next.js) first, because the portfolio site is the first real project. Laravel/PHP comes second. The Gate scripts are stack-agnostic and read a per-project configuration.
- **C42. Coding conventions.**
  - **Mechanical rules are Gates, not skill prose:** linter, formatter and strict type-checker configuration in the template.
  - **Skills carry only judgment rules** that no tool can check, such as domain-first structure and how tests should be written.
- **C43. Source for the Laravel/PHP conventions.** They are inspired by the owner's employer's internal guidelines, which are under K2 and must not be copied. Those guidelines appear to follow a publicly published Laravel/PHP style guide closely. The harness's own PHP conventions should cite and build on that public source. This is to be verified when the PHP template is built.

### Facts found in the docs

- **Workflow scripts cannot run commands or read files.** Only the agents they start can. This leads to [ADR-0006](adr/0006-ci-is-the-binding-gate.md): CI is the binding Gate, and Gates inside a Run are an inner loop.
- **A plugin cannot ship sandbox or permission settings.** This is not documented. Those settings therefore live in the template's `.claude/settings.json`, which resolves an open fact from the revised summary.
- **The plugin's `bin/` executables are put on the agent's PATH.** The Gate scripts ship there.
