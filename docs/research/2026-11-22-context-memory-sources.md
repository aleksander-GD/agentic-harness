# Review of the owner's context and memory sources

- **Date:** 2026-11-22
- **Method:** web pages read by a fetch agent. Some Claude doc pages came back truncated or summarised; this is flagged where it matters.
- **Status:** research input, used to check decision C20 (curated memory).

Each statement carries a tag:

- **[documented]**: the source says it, with evidence or as official documentation.
- **[claimed]**: the source asserts it without evidence.
- **[inference]**: our own interpretation.

## 1. Moderne: "Moderne agent tools" (https://moderne.ai/moderne-agent-tools)

**Type.** A product page for a paid platform. The vendor's headline is "match the tool to the shape of the problem". [documented]

**Parts.** [documented]

- *Recipes*: deterministic code transformations.
- *Trigrep*: an indexed code search.
- *Prethink*: a "precomputed context layer". Static analysis builds architecture and dependency graphs before a session starts.
- All of it is delivered over MCP.

**Claims.**

- "A Java 8 to 25 migration drops from 61 million tokens to 30,000". [claimed] The page gives no method; a linked benchmark was not reviewed.
- "More than 7,200 recipes". [claimed]
- Trigrep answers "in under a second with no follow-up reads required". [claimed]

**Preconditions.** It is centred on Java and aimed at enterprise multi-repository fleets. The page concedes, "For one repo, yes." [documented]

**Relevance.** It argues *for* generated memory, but at fleet scale and for deterministic migrations. The 61M-token figure compares an agent hand-editing thousands of files with a deterministic transformation, which does not carry over to single-repository TDD work. [inference]

The idea worth keeping: use deterministic tools for deterministic work. The harness already does this with its Gates and scripts. [inference]

## 2. Gist: "Harness Engineering Best Practices from Anthropic and OpenAI" (celesteanders, 2026-04-02)

**Type.** A personal compilation of other people's posts. It is second-hand, and it gives no evidence of its own. [documented]

**Recommendations.** [claimed]

- Start with a loop around a single agent.
- Persist state in files: task lists, progress notes, specs, git history.
- Keep CLAUDE.md to about 100 lines and treat it as "a map, not an encyclopedia".
- Use a fresh context for each session.
- Use tests and linters as "backpressure".
- Commit after each task.
- Use separate planner, generator and evaluator roles, because agents "rate their own work too generously".
- Re-evaluate the harness after each model upgrade.

**Preconditions.** It targets long-running, unattended app building, which is heavier than an interactive solo MVP. [inference]

**Where it agrees with decisions so far.** [inference]

- Single agent (C11).
- Files as memory (C20).
- Tests as the arbiter (Gates).
- An independent reviewer (H3).

**Where it differs.** It suggests letting the agent update CLAUDE.md itself, which conflicts with C20's curated memory. It also treats *task progress state* and *project knowledge* without separating them. [inference]

## 3. Claude Developer Platform docs (https://platform.claude.com/docs/en/)

### Context rot

The docs state: "As token count grows, accuracy and recall degrade, a phenomenon known as *context rot*." Curating what goes into context matters "just as much as" how much space is available. [documented: context-windows]

Current models have a 1M-token window. [documented: models overview]

### Context management

- For long agentic work, the docs recommend server-side compaction as "the primary strategy". [documented]
- Context editing can clear old tool results. [documented]

### Memory tool

- It runs on the client side: memory is files that the application stores. [documented]
- The guidance is to periodically delete files nobody has read in a long time. [documented]
- There is **no guidance on keeping remembered facts current**. [documented absence]

### Choosing a model

- The docs say "start with Claude Opus 5.5 for most workloads" if unsure. [documented]
- They also say "tuning effort is often a better lever than switching models". [documented]
- Haiku 5.5 is listed for the lowest latency and price, with "subagent tasks" given as an example. [documented]

List prices per million input/output tokens: [documented]

| Model | Input | Output |
|---|---|---|
| Opus 5.5 | $4 | $20 |
| Sonnet 5.5 | $2 | $10 |
| Haiku 5.5 | from $0.10 | from $0.50 |

### Prompt caching

**Prices.** [documented]

- A cache read costs 0.1× the base input price, or 0.05× on Opus 5.5 and Sonnet 5.5.
- A cache write costs 1.25× for the default 5-minute lifetime, or 2× for a 1-hour lifetime.

**What breaks the cache.** Content is cached in the order tools → system → messages. Changing the tool definitions invalidates the whole cache. [documented]

### Tool search

- Accuracy at picking the right tool "degrades once you exceed 30–50 available tools". [documented]
- The docs recommend tool search once there are 10 or more tools. [documented]

## Verdict on C20 (curated memory)

C20 holds.

- **Documented evidence supports it:**
  - context rot argues for small, relevant context;
  - the docs give no way to keep memory current.
- **The one source that favours generated memory** is a vendor talking about fleets of enterprise repositories. [inference]

### Refinements surfaced

1. **Separate project knowledge from task state.** [inference]
   - *Project knowledge* is durable, curated and reviewed: the glossary, ADRs and Behaviour Lists.
   - *Task state* is short-lived and written by the agent: progress notes and what is left to do in the current task.
   - The gist's advice covers task state. C20 covers project knowledge.
2. **Should the agent be allowed to propose edits to project knowledge?** This is open; see Round 8.
3. **Keep CLAUDE.md short for quality, not cost.** [inference]
   - Stable instructions are cached, so re-reading them every turn costs only 5–10% of the normal input price.
   - Shrinking CLAUDE.md therefore saves little money.
   - It still matters, because every line takes space in the context window, and context rot is documented.
