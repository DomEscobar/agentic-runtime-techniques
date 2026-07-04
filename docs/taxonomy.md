# Taxonomy

Agentic runtime techniques can be grouped by what they make reliable.

## 1. Core Action Loops

The basic model-tool-environment loop.

Canonical shape:

```text
observe -> reason/plan -> act/tool call -> observe result -> continue or stop
```

Examples:

- ReAct
- Toolformer-style tool use
- Function-calling agent loops

Main value:

- Gives the model a structured way to interact with an environment.

Main risk:

- Long contexts drift. Stop conditions are often weak.

## 2. Planning Loops

The system decomposes work into steps, then executes and updates the plan.

Canonical shape:

```text
plan -> execute step -> observe -> revise plan -> execute next step
```

Examples:

- Plan-and-Execute
- ReWOO
- task decomposition agents

Main value:

- Makes larger tasks tractable.

Main risk:

- Plans can become stale or overfit to early assumptions.

## 3. Verification Loops

The system runs work against an objective and uses a checker to decide whether
to continue.

Canonical shape:

```text
attempt -> verify -> fix or finish
```

Examples:

- Codex goals
- reviewer/QA agent loops
- test-driven coding agents

Main value:

- Creates a termination condition outside the worker agent.

Main risk:

- The verifier becomes the weak link.

## 4. Bounded Retry Loops

Execution is bounded by steps, budget, time, or context. Failure is explicit and
used as signal.

Canonical shape:

```text
attempt with bounds -> collect result -> retry/fresh context/escalate
```

Examples:

- Ralph Loop
- context-reset coding loops
- budget-limited subagent loops

Main value:

- Prevents infinite agent wandering and makes failure operationally useful.

Main risk:

- Too much bounding can discard useful intermediate context.

## 5. Reflection and Memory Loops

The agent critiques its output or updates memory for later runs.

Canonical shape:

```text
act -> reflect -> store lesson -> retry or use later
```

Examples:

- Reflexion
- self-critique loops
- episodic memory update loops

Main value:

- Improves future attempts without changing model weights.

Main risk:

- Bad reflections can pollute memory.

## 6. Research Loops

The system searches, reads, synthesizes, and then searches again.

Canonical shape:

```text
question -> queries -> read -> synthesize -> identify gaps -> next queries
```

Examples:

- OpenAI Deep Research style workflows
- literature-review agents
- source-grounded answer loops

Main value:

- Handles open-ended information gathering.

Main risk:

- Source quality and citation discipline determine output quality.

## 7. Experiment Loops

The system proposes variants, runs them, evaluates results, and keeps or
reverts changes.

Canonical shape:

```text
propose experiment -> run -> evaluate -> keep/revert -> next experiment
```

Examples:

- Autoresearch
- automated ML/code experiment loops
- hillclimbing agents

Main value:

- Turns open-ended improvement into a measurable search process.

Main risk:

- Metrics can be gamed or too narrow.

## 8. Multi-agent Orchestration Loops

Multiple agents have fixed or dynamic roles and communicate through channels,
state, queues, or artifacts.

Canonical shape:

```text
planner -> task queue -> workers -> reviewer -> merge/report
```

Examples:

- planner/executor harnesses
- debate/council agents
- swarm-style task routing

Main value:

- Parallelizes work and separates responsibilities.

Main risk:

- Coordination overhead can exceed the benefit.

## 9. Durable Runtime Loops

The agent system behaves like a long-running service.

Canonical shape:

```text
load state -> work/check -> checkpoint -> sleep/wait -> resume
```

Examples:

- heartbeat agents
- checkpoint loops
- task queues and cron-backed agents
- persistent typed state runtimes

Main value:

- Makes agents operational rather than one-shot.

Main risk:

- Durability, state migration, and lifecycle semantics become real software
  engineering problems.

## 10. Coding Harness Loops

Agent execution is wrapped in source-control, tests, review, and rollback.

Canonical shape:

```text
branch/worktree -> implement -> test -> review -> fix -> merge or revert
```

Examples:

- implement-review-merge loops
- PR comment response loops
- worktree-isolated coding agents

Main value:

- Makes code changes auditable and reversible.

Main risk:

- Test gaps and reviewer blind spots still leak defects.

## Cross-Cutting Runtime Layers

Categories 1-10 describe the shape of an agent's reasoning loop. The
categories below are not reasoning loops themselves; they are runtime layers
that wrap, gate, extend, or observe any of the loops above. A production
system usually composes one loop-shape category with several of these
layers.

### 11. Test-Time Compute / Search Loops

The system spends extra inference compute at run time, sampling or expanding
multiple candidates and searching for the best one, instead of committing to
a single pass.

Canonical shape:

```text
sample/expand candidates -> score or evaluate -> select or backtrack -> repeat until budget exhausted
```

Examples:

- Self-Consistency / sample-and-vote
- Tree of Thoughts, Graph of Thoughts
- Language Agent Tree Search (LATS)
- Verifier/PRM-guided step search

Main value:

- Trades extra inference compute for higher answer reliability on hard
  reasoning tasks.

Main risk:

- Cost scales quickly with sample or branch count; needs a hard compute
  budget.

### 12. Human-in-the-Loop and Governance Loops

Risky or uncertain actions pause for a human decision instead of proceeding
automatically.

Canonical shape:

```text
proposed action -> risk policy -> interrupt -> human decision -> resume
```

Examples:

- HITL Interrupt / Approval Gate
- Evaluator/Optimizer Loop with HITL fallback

Main value:

- Makes human review a first-class, resumable runtime branch instead of an
  ad hoc chat aside.

Main risk:

- Approval fatigue and miscalibrated policy thresholds.

### 13. Runtime Security and Access Control

The system decides what an agent is allowed to see, call, or spend,
independent of how well it reasons.

Canonical shape:

```text
untrusted input/tool -> trust classification/capability check -> screen or scope -> allow/block/escalate
```

Examples:

- Prompt-Injection Action Firewall
- Context Trust Zones
- Capability / Least-Privilege Runtime
- Runtime Budget Policy Engine

Main value:

- Bounds agent authority and blast radius even when reasoning fails.

Main risk:

- Over-restriction blocks legitimate work; under-restriction leaves
  excessive agency.

### 14. Context and Memory Runtime

The system treats context and memory as scarce, managed resources rather
than an ever-growing transcript.

Canonical shape:

```text
sources -> rank/filter/pack -> budget check -> summarize or page out -> model -> update memory
```

Examples:

- Context Packing / Compression Loop
- MemGPT-style memory paging
- Episodic vector-store memory retrieval
- Skill Library / lifelong skill acquisition

Main value:

- Keeps long-running agents within context limits while preserving
  decisive information across turns and sessions.

Main risk:

- Compression or paging can silently drop what turns out to be the
  important detail.

### 15. Harness and Composition Runtime

The system separates a reusable "agent brain" from the UI, session, and
persistence concerns that surround it.

Canonical shape:

```text
caller/session -> harness (transcript, queues, cancellation) -> stateless loop -> events -> caller/session
```

Examples:

- Stateful Harness Around Pure Agent Loop
- DeerFlow-style middleware stack
- Workflow-graph / state-machine runtime

Main value:

- Lets the same core loop power a CLI, TUI, API, or test harness without
  rewriting it.

Main risk:

- Ownership boundaries blur if the harness and loop both try to own state.

### 16. Protocol and Interop Layers

Tools and remote agents become portable across frameworks through a shared
protocol instead of bespoke integrations.

Canonical shape:

```text
host/client -> capability discovery -> invoke tool/agent -> status/result contract
```

Examples:

- MCP Tool / Context Protocol Layer
- A2A Agent Interoperability Protocol Layer

Main value:

- Makes tools and remote agents reusable across frameworks and vendors.

Main risk:

- Protocol plumbing can be mistaken for agent reasoning; host-side
  authorization is still required.

### 17. Observability and Provenance Loops

The system emits structured records of what happened so runs can be
debugged, audited, and replayed after the fact.

Canonical shape:

```text
run -> structured trace/provenance events -> inspect/evaluate/replay/audit
```

Examples:

- Run Receipt / Trace-First Runtime
- Artifact Provenance Graph
- Append-Only Session Event Log / Branchable Replay

Main value:

- Makes agent failures diagnosable and outputs auditable.

Main risk:

- Traces without redaction can leak private context; logs without replay
  are only half useful.

### 18. Cost, Latency, and Serving Runtime

The system treats inference spend and serving efficiency as an explicit
runtime concern, not an incidental property of whichever model or workflow a
request happens to hit.

Canonical shape:

```text
request/query -> route/place/cache decision against cost-quality tradeoff -> cheap path or escalate -> execute -> record cost
```

Examples:

- Model Cascading / Tiered Routing
- Semantic / Prompt Caching Layer
- Cost-Aware Agentic Query Optimizer
- Agent-Aware Serving Policy Layer

Main value:

- Keeps inference spend and latency proportional to task difficulty instead
  of paying premium-model, premium-compute prices for every request.

Main risk:

- Misrouted or misclassified requests silently trade away quality for cost,
  and the failure only shows up downstream.
