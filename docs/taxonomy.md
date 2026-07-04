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
