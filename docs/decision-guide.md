# Decision Guide

Status: 2026-07-04

Use this guide when designing an agentic system. The aim is not to pick the
flashiest pattern; it is to choose the smallest runtime that makes the system
reliable for its job.

## Quick Selection

| Need | Start With | Add When Needed | Avoid When |
| --- | --- | --- | --- |
| Simple tool-using assistant | ReAct / core tool loop | verifier, loop breaker, context packing | task is long-running or high-risk |
| Reusable interactive agent brain | Stateful harness around pure loop | steering queues, cancellation, event stream | one-shot stateless API call |
| Long task with visible progress | Plan-and-execute | todo state, verifier, status contract | task can be answered in one pass |
| Correctness matters | Verifier / goal loop | judge, tests, rubric, typed blockers | no objective check exists |
| Open-ended research | Research loop | source ledger, gap analysis, citation verifier | sources are unavailable or untrusted |
| Coding changes | Coding harness loop | tests, reviewer, worktree isolation, rollback | repo has no runnable checks |
| Many domains or specialists | Supervisor | typed worker results, delegation ledger, forward_message | one specialist can do the task |
| Low-friction specialist handoff | Swarm | active-agent state, max turns, last-agent memory | global plan/governance is critical |
| Loose collaboration | Blackboard | claim/evidence schema, arbiter, freshness rules | shared state cannot be trusted |
| Risky external actions | HITL approval gate | checkpoint, audit trail, edit/reject branch | action is harmless and reversible |
| Long-running background work | Task queue / heartbeat | checkpoint, retry/backoff, status events | synchronous answer is enough |
| Interop with external tools | MCP layer | tool policy, schema validation, tracing | all tools are local and fixed |
| Interop with remote agents | A2A layer | auth, agent cards, task artifacts | local tools/subagents are enough |
| Production debugging/audit | Run receipts / tracing | redaction, replay, support bundle | no state/tool calls are involved |
| Hostile or untrusted context | Prompt-injection action firewall | trust zones, least privilege, HITL | no untrusted context enters the run |
| Powerful tools/secrets | Capability runtime | scoped grants, expiry, audit | no external authority is available |
| Expensive/long autonomous runs | Budget policy engine | warn thresholds, hard stops, checkpoint | bounded single-turn answer |
| Auditable artifacts/claims | Provenance graph | claim IDs, source IDs, redaction | output is disposable |
| Durable branchable sessions | Append-only session log | replay, leaf pointer, compaction entries | no resume/branch/audit needed |

## Design From Requirements

### If the system must use tools

Base pattern:

- ReAct / core tool loop

Required runtime primitives:

- tool schemas
- observations
- stop condition
- loop detection
- token/cost budget
- tool error recovery

Upgrade path:

- Add a verifier loop when outputs need correctness checks.
- Add a guardrail pipeline when tools can access sensitive data or take action.
- Add MCP when tools should be provided by external servers.

Do not stop at "the model can call tools." A production tool loop needs policy,
error handling, observability, and termination.

### If the system needs a reusable interactive agent brain

Base pattern:

- Stateful harness around pure agent loop

Required runtime primitives:

- caller-owned transcript
- stateless provider/tool loop
- provider-neutral event stream
- single-runner invariant
- steering and follow-up queues
- cancellation token
- interrupted tool-call repair

Upgrade path:

- Add durable session storage outside the harness.
- Add run receipts or tracing from the event stream.
- Add task queue semantics if runs can outlive the frontend process.

Main decision:

- If the agent is an app or frontend surface, use a harness.
- If the agent is only a one-shot API call, a harness may be unnecessary.
- If cancellation is possible, repair dangling tool calls before the next model
  request.

### If the system must finish multi-step work

Base pattern:

- Plan-and-execute

Required runtime primitives:

- plan artifact
- mutable task/todo state
- one active step or explicit parallel branches
- progress status
- stale-plan detection

Upgrade path:

- Add verifier/goal loop for done checks.
- Add task queue/resumable workflow if the work can outlive one chat turn.
- Add a reviewer or judge when quality matters.

Main decision:

- If the plan is small and stable, keep it in one agent.
- If the plan has independent work streams, use planner/executor or supervisor.
- If the plan is mostly uncertain exploration, use research or experiment loops.

### If correctness matters more than speed

Base pattern:

- Verifier / goal loop

Required runtime primitives:

- explicit goal
- acceptance criteria
- verifier or tests
- typed blocker states
- retry budget

Upgrade path:

- Add agent-as-judge for rubric-based review.
- Add multi-agent debate only when independent reasoning diversity is likely to
  help.
- Add run receipts so failures can be inspected later.

Anti-pattern:

- Do not let the same worker silently decide it is done if the cost of being
  wrong is high.

### If the system researches or answers with facts

Base pattern:

- Research loop

Required runtime primitives:

- query planner
- source extraction
- source ledger
- synthesis step
- gap analysis
- citation/factuality check

Upgrade path:

- Add Self-RAG / corrective RAG when retrieval quality is uncertain.
- Add context packing when many sources exceed the prompt budget.
- Add run receipts when citations or auditability matter.

Anti-pattern:

- Do not treat "used search" as evidence. Store the sources, claims, and where
  each claim came from.

### If the system edits code

Base pattern:

- Coding harness loop

Required runtime primitives:

- repo inspection
- edit isolation or clean diff discipline
- tests/checks
- reviewer or verifier
- failure report

Upgrade path:

- Add SWE-agent/ACI-style interfaces when the coding environment matters.
- Add implement/review/merge when changes are risky.
- Add Ralph-style bounded retry for long autonomous attempts.

Anti-pattern:

- Do not use a generic chat loop for code without a test or review boundary.

### If many agents are involved

Choose by coordination shape:

| Situation | Pattern |
| --- | --- |
| Need central governance | Supervisor |
| Need direct specialist handoff | Swarm |
| Need multiple levels of ownership | Hierarchical supervisor |
| Need parallel specialists with shared evidence | Blackboard |
| Need independent reasoning and aggregation | Debate / committee |
| Need remote third-party agents | A2A protocol layer |

Required runtime primitives:

- role/capability registry
- delegation or handoff contract
- max turns / max depth
- result schema
- no-progress breaker
- trace of who did what

Anti-pattern:

- Do not add agents just to make the architecture sound agentic. Add agents
  when separate context, tools, authority, or evaluation improves the system.

### If the system runs in production

Base patterns:

- Task queue / resumable workflow
- HITL approval gate
- Guardrail pipeline
- Run receipt / trace-first runtime
- Context packing / compression loop
- Prompt-injection action firewall
- Capability / least-privilege runtime
- Runtime budget policy engine
- Artifact provenance graph
- Append-only session event log

Required runtime primitives:

- task IDs
- checkpoints
- status events
- retry/backoff
- policy gates
- scoped capabilities
- budget thresholds
- audit trail
- trace redaction
- provenance links
- support bundle or replay path

Anti-pattern:

- Do not build a long-running production agent as one synchronous chat turn.

## Composition Recipes

### Reliable Tool Assistant

```text
Intent router
  -> stateful harness
  -> ReAct tool loop
  -> tool error recovery
  -> loop/token breaker
  -> verifier
  -> trace receipt
```

Use when:

- The assistant calls tools but should stay responsive and bounded.

### Deep Research Agent

```text
Research loop
  -> source ledger
  -> context packing
  -> gap analysis
  -> citation verifier
  -> final synthesis
```

Use when:

- The answer depends on external, changing, or disputed facts.

### Coding Agent

```text
Repo inspection
  -> stateful harness
  -> plan/todos
  -> edit loop
  -> tests
  -> reviewer/verifier
  -> commit or handoff
```

Use when:

- The output is a code change, not just an explanation.

### Multi-Agent Operations System

```text
Intent router
  -> supervisor or swarm
  -> worker agents
  -> typed results
  -> verifier
  -> run receipt
```

Use when:

- Different roles need different tools, context, or authority.

### Durable Background Agent

```text
Task queue
  -> worker lease
  -> checkpoint/status
  -> HITL interrupt if risky
  -> retry/resume
  -> final artifact
```

Use when:

- Work can take minutes, hours, or multiple wakeups.

## Decision Questions For Agents

Before choosing a pattern, answer these:

1. Is this one-shot, multi-step, or long-running?
2. Can success be verified objectively?
3. Are external actions reversible or risky?
4. Does the task require tools, retrieval, code execution, or remote agents?
5. Does one agent have enough context and authority?
6. What state must survive context loss, crash, or handoff?
7. What should happen on no progress, budget exhaustion, or tool failure?
8. What trace would a human need to debug the run?

## Failure Mode Map

| Failure Mode | Add This |
| --- | --- |
| Infinite tool loop | loop detector, token budget, Ralph-style bound |
| Fake completion | verifier loop, tests, typed blockers |
| Stale plan | mutable plan state, replan trigger |
| Bad retrieval | Self-RAG / corrective RAG, source quality scoring |
| Context overflow | context packing, summarization, provenance |
| Unsafe tool action | HITL gate, guardrail pipeline |
| Multi-agent drift | delegation ledger, max handoffs, result schemas |
| Hidden production failure | run receipts, tracing, status events |
| Crash or long wait | task queue, checkpoint, retry/backoff |
| User cannot audit result | source ledger, artifacts, trace summary |

## Minimal Viable Runtime

For most useful systems, the minimum is:

```text
intent/context router
  -> bounded tool loop or plan loop
  -> verifier / done check
  -> error recovery
  -> trace or run receipt
```

For production systems, add:

```text
task IDs
checkpoints
status events
approval gates
guardrails
redacted traces
```

If a design has none of these, it is probably just a prompt wrapped around a
model call, not an agentic runtime.
