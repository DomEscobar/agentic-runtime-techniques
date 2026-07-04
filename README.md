# Agentic Runtime Techniques

A field guide to the loops, harnesses, orchestration patterns, and runtime
primitives used to build reliable agent systems.

This repo exists because "agentic loop" has become an overloaded phrase. Some
people mean the classic model-tool-observation loop. Others mean a verifier
loop, a planner/executor harness, a long-running heartbeat, a task queue, or a
durable runtime with typed state. Those are related ideas, but they solve
different problems.

The goal is to collect the techniques, name them clearly, compare their trade
offs, and link to papers, repos, posts, and production implementations.

## Scope

Included:

- Agent execution loops
- Planning and decomposition loops
- Verification, review, and goal loops
- Reflection and memory loops
- Research and experiment loops
- Multi-agent orchestration harnesses
- Durable runtime patterns such as heartbeat, checkpoint, task queues, and typed
  state
- Coding-agent harnesses such as worktree isolation, reviewer agents, rollback,
  and PR comment loops

Not included by default:

- Generic prompt engineering tips unless they define an execution pattern
- One-off agent products without a reusable runtime technique
- Marketing-only framework pages with no concrete loop, primitive, or harness

## Start Here

- [Taxonomy](docs/taxonomy.md)
- [ASCII flow diagrams](docs/flow-diagrams.md)
- [Benchmark/date tier list](docs/tier-list.md)
- [OpenClaw and Hermes technique notes](docs/openclaw-hermes-techniques.md)
- [DeerFlow technique notes](docs/deer-flow-techniques.md)
- [Supervisor and swarm technique notes](docs/supervisor-swarm-techniques.md)
- [Machine-readable catalog](data/techniques.yml)
- [Contribution guide](CONTRIBUTING.md)

## Initial Categories

| Category | Core Question |
| --- | --- |
| Core action loops | How does one agent decide, act, observe, and continue? |
| Planning loops | How is a task decomposed before or during execution? |
| Verification loops | How does the system decide whether work is done? |
| Reflection loops | How does the agent critique or update itself? |
| Research loops | How does the agent search, synthesize, and deepen a topic? |
| Experiment loops | How does the agent try variants and keep/revert outcomes? |
| Multi-agent loops | How do multiple agents coordinate work? |
| Durable runtime loops | How do agents persist, sleep, resume, checkpoint, and expose state? |
| Coding harness loops | How do agents edit code safely and verify changes? |

## Seed Techniques

The first catalog pass includes ReAct, Plan-and-Execute, Reflexion, Ralph Loop,
RLM/CodeAct-style REPL loops, Deep Research, Codex Goal/verifier loops,
Autoresearch, planner/executor harnesses, heartbeat/checkpoint loops, and
implement-review-merge coding loops.

## Naming Rule

Prefer the most specific useful name:

- Use **ReAct** for think-act-observe loops.
- Use **verifier loop** when the key primitive is objective checking.
- Use **Ralph Loop** when the key primitive is bounded fresh-context execution.
- Use **heartbeat/checkpoint loop** when the key primitive is durable periodic
  execution with status emission.
- Use **planner/executor harness** when fixed roles coordinate many workers.

## License

MIT for repo content unless a linked source states otherwise.
