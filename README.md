# Agentic Runtime Techniques

A field guide to the loops, harnesses, orchestration patterns, and runtime
primitives used to build reliable agent systems.

This repo exists because "agentic loop" has become an overloaded phrase. Some
people mean the classic model-tool-observation loop. Others mean a verifier
loop, a planner/executor harness, a long-running heartbeat, a task queue, or a
durable runtime with typed state. Those are related ideas, but they solve
different problems.

The goal is to help agents and builders choose runtime designs. The repo should
answer:

- What kind of agentic system am I building?
- Which runtime pattern should I start from?
- Which supporting primitives do I need?
- What failure modes should I guard against?
- What evidence or benchmark signal exists?

The repo collects the techniques, names them clearly, compares their tradeoffs,
links to evidence, and provides machine-readable discovery data.

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

- [Decision guide](docs/decision-guide.md)
- [Taxonomy](docs/taxonomy.md)
- [ASCII flow diagrams](docs/flow-diagrams.md)
- [Benchmark/date tier list](docs/tier-list.md)
- [OpenClaw and Hermes technique notes](docs/openclaw-hermes-techniques.md)
- [DeerFlow technique notes](docs/deer-flow-techniques.md)
- [Tau harness technique notes](docs/tau-harness-techniques.md)
- [Security and governance runtime patterns](docs/security-governance-patterns.md)
- [Supervisor and swarm technique notes](docs/supervisor-swarm-techniques.md)
- [Additional runtime patterns](docs/additional-runtime-patterns.md)
- [2026 emerging runtime patterns](docs/2026-emerging-patterns.md)
- [Machine-readable catalog](data/techniques.yml)
- [Agent discovery index](data/agent-discovery-index.yml)
- [Runtime analyses](analysis/)
- [2026 arXiv runtime technique sweep](analysis/2026-arxiv-agentic-runtime-techniques.md)
- [Contribution guide](CONTRIBUTING.md)

## How An Agent Should Use This Repo

1. Start with [Decision guide](docs/decision-guide.md) and identify the
   system goal: answer questions, use tools, research, code, delegate, run
   in the background, or govern risky actions.
2. Select a base control loop from [Taxonomy](docs/taxonomy.md).
3. Add required runtime primitives from the decision guide: state, verifier,
   checkpoints, approval gates, guardrails, traces, task queues, or handoffs.
4. Inspect [ASCII flow diagrams](docs/flow-diagrams.md) to understand branches,
   retries, stop conditions, and human-in-the-loop points.
5. Use [Benchmark/date tier list](docs/tier-list.md) to understand evidence
   strength before claiming a pattern is proven.
6. Read the implementation notes for production examples:
   [OpenClaw/Hermes](docs/openclaw-hermes-techniques.md),
   [DeerFlow](docs/deer-flow-techniques.md),
   [Tau](docs/tau-harness-techniques.md),
   [Security/Governance](docs/security-governance-patterns.md),
   [Supervisor/Swarm](docs/supervisor-swarm-techniques.md),
   [Additional runtime patterns](docs/additional-runtime-patterns.md), and
   [2026 emerging runtime patterns](docs/2026-emerging-patterns.md).

For automated consumption, use `data/agent-discovery-index.yml` first, then
follow its links into `data/techniques.yml` and the docs.

## Initial Categories

Categories 1-10 describe the shape of an agent's reasoning loop. Categories
11-18 are cross-cutting runtime layers that wrap, gate, extend, or observe
any of those loops; see [Taxonomy](docs/taxonomy.md) for the full writeup of
each one.

| Category | Core Question |
| --- | --- |
| 1. Core action loops | How does one agent decide, act, observe, and continue? |
| 2. Planning loops | How is a task decomposed before or during execution? |
| 3. Verification loops | How does the system decide whether work is done? |
| 4. Bounded retry loops | How is agent execution kept from wandering forever? |
| 5. Reflection and memory loops | How does the agent critique or update itself across attempts? |
| 6. Research loops | How does the agent search, synthesize, and deepen a topic? |
| 7. Experiment loops | How does the agent try variants and keep/revert outcomes? |
| 8. Multi-agent orchestration loops | How do multiple agents coordinate work? |
| 9. Durable runtime loops | How do agents persist, sleep, resume, checkpoint, and expose state? |
| 10. Coding harness loops | How do agents edit code safely and verify changes? |
| 11. Test-time compute / search loops | How does the system spend extra inference compute to search for a better answer? |
| 12. Human-in-the-loop and governance loops | How do risky actions pause for a human decision? |
| 13. Runtime security and access control | What is an agent allowed to see, call, or spend? |
| 14. Context and memory runtime | How is context and memory managed as a scarce resource across turns and sessions? |
| 15. Harness and composition runtime | How is the reusable agent brain separated from session, UI, and persistence concerns? |
| 16. Protocol and interop layers | How do tools and remote agents stay portable across frameworks? |
| 17. Observability and provenance loops | How do runs become debuggable, auditable, and replayable? |
| 18. Cost, latency, and serving runtime | How is inference spend and serving efficiency managed as an explicit runtime concern? |

## Seed Techniques

`data/techniques.yml` is the single, complete canonical catalog of every
technique in this repo (79 entries as of this pass), each with a summary,
loop shape, runtime primitives, tradeoffs, and sourced references. The
per-domain deep-dive docs (`docs/openclaw-hermes-techniques.md`,
`docs/deer-flow-techniques.md`, `docs/tau-harness-techniques.md`,
`docs/security-governance-patterns.md`, `docs/supervisor-swarm-techniques.md`,
`docs/additional-runtime-patterns.md`) remain useful companions with more
implementation detail, but every technique they describe also has a
canonical entry in `data/techniques.yml`.

The catalog spans: core loops (ReAct, Plan-and-Execute, ReWOO, Reflexion,
Ralph Loop, RLM/CodeAct REPL, Deep Research, Codex Goal/Verifier Loop,
Autoresearch); multi-agent orchestration (Planner/Executor Harness,
Supervisor, Swarm, Hierarchical Supervisor Tree, Blackboard, Debate,
Agent-as-Judge, Contract-Net delegation); test-time compute and search
(Self-Consistency, Tree/Graph of Thoughts, Language Agent Tree Search,
PRM-guided step search); retrieval and query techniques (Self-Ask, HyDE,
Self-RAG, Corrective RAG, Toolformer, Least-to-Most Prompting); memory
architecture (MemGPT-style paging, episodic vector-store retrieval, Skill
Library); cost and latency runtime (Model Cascading, Semantic/Prompt
Caching); transactional safety (Saga/Compensating Transactions, Idempotency
Keys); agent evaluation harnesses (Offline Regression Eval, Adversarial
Red-Team loop); runtime primitives (MCP, A2A, HITL, Guardrail Pipeline,
Run Receipts, Context Packing, Task Queue, Stateful Harness,
Prompt-Injection Firewall, Capability Runtime, Budget Policy Engine, Tool
Reliability Scoring, Provenance Graph, Append-Only Session Log,
Workflow-Graph Runtime); and a 2026 emerging-patterns family from a
DeepResearch arXiv sweep (Infinite Agentic Loop Static Analysis,
Self-Healing Orchestrator, Workflow Portfolio Router, Meta-Tool Compiler,
Cost-Aware Agentic Query Optimizer, Trajectory-Level Reward/Verifier,
Memory Write-Manage-Read Loop, Agent-Aware Serving Policy Layer), detailed
in [docs/2026-emerging-patterns.md](docs/2026-emerging-patterns.md).

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
