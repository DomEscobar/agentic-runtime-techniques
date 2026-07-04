# Additional Agentic Runtime Patterns

Status: 2026-07-04

This file collects runtime patterns that are distinct from the classic
single-agent tool loop and from supervisor/swarm orchestration. These are worth
tracking because they change control flow, state ownership, safety boundaries,
or interop contracts.

## 1. Blackboard / Shared Workspace Coordination

```text
goal -> blackboard -> agents watch/post claims -> arbiter/synthesizer -> answer
```

Multiple specialist agents coordinate through a shared artifact rather than
through direct chat or a single supervisor message chain. The blackboard stores
requests, partial findings, claims, evidence, conflicts, and final artifacts.

Runtime primitives:

- shared blackboard state
- capability-based agent subscription
- claim/evidence records
- conflict markers
- arbiter or synthesizer
- freshness/version controls

Why it matters:

- Good for loose, opportunistic collaboration.
- Reduces direct agent-to-agent coupling.
- Makes partial work inspectable.

Risks:

- Stale claims can accumulate.
- Needs ownership rules or the board becomes noisy.
- Arbitration can become a hidden supervisor.

Sources:

- Google Research, "Blackboard Multi-Agent Systems for Information Discovery in
  Data Science": https://research.google/pubs/blackboard-multi-agent-systems-for-information-discovery-in-data-science/
- AWS DevOps, "Multi Agent Collaboration with Strands":
  https://aws.amazon.com/blogs/devops/multi-agent-collaboration-with-strands/

## 2. Multi-Agent Debate / Committee Deliberation

```text
question -> N agents answer -> debate rounds -> judge/vote/stability check -> final
```

Several agents produce independent answers, then critique or debate each other
over one or more rounds before a final judge, vote, or convergence check.

Runtime primitives:

- independent proposer agents
- debate transcript
- round budget
- judge, voter, or aggregation policy
- stability/convergence criterion

Why it matters:

- Useful when diversity of reasoning matters.
- Can expose contradictions before final output.
- Works as an evaluation/runtime pattern, not only as answer generation.

Risks:

- Token and latency cost scale quickly.
- Weak agents can reinforce wrong consensus.
- Majority pressure can suppress correction.

Sources:

- "Improving Factuality and Reasoning in Language Models through
  Multiagent Debate": https://arxiv.org/abs/2305.14325
- "Can LLM Agents Really Debate? A Controlled Study of Multi-Agent Debate in
  Logical Reasoning": https://arxiv.org/abs/2511.07784

## 3. Agent-as-Judge / Referee Team

```text
candidate output -> judge agent(s) -> rubric checks -> accept / revise / reject
```

An evaluator agent, or a small referee team, scores candidate outputs against a
rubric and routes the workflow to accept, revise, reject, or escalate.

Runtime primitives:

- rubric
- candidate artifact
- judge agent or judge panel
- score and rationale schema
- acceptance threshold
- revision route

Why it matters:

- Makes quality gates explicit.
- Can be attached to coding, research, support, or extraction workflows.
- Bridges evaluation and runtime control.

Risks:

- Judge bias can become a single point of failure.
- Rubrics need calibration.
- Self-judging can overestimate correctness.

Sources:

- "Enhancing LLM-as-a-Judge via Multi-Agent Collaboration":
  https://cdn.amazon.science/48/5d/20927f094559a4465916e28f41b5/enhancing-llm-as-a-judge-via-multi-agent-collaboration.pdf
- "Multi-Agent Debate for LLM Judges with Adaptive Stability Detection":
  https://arxiv.org/html/2510.12697v1

## 4. A2A / Agent Interoperability Protocol Layer

```text
agent client -> discover agent card -> send/stream task -> poll/status/artifacts
```

Agent-to-agent protocols make remote agents invocable as durable task endpoints
instead of local in-process tools. The runtime boundary becomes a protocol:
discovery, auth, task IDs, streaming, status, and artifacts.

Runtime primitives:

- agent card / capability manifest
- authenticated endpoint
- task ID
- send / stream / poll operations
- status and artifact contract

Why it matters:

- Lets agents from different frameworks coordinate.
- Supports long-running remote work.
- Encourages typed task status instead of opaque chat messages.

Risks:

- Protocol support is still moving.
- Trust, auth, and data minimization become central.
- Remote agents may hide implementation and evaluation details.

Sources:

- Google, "Announcing the Agent2Agent Protocol":
  https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/
- Google Codelab, A2A agent cards:
  https://codelabs.developers.google.com/intro-a2a-purchasing-concierge

## 5. MCP Tool/Context Protocol Layer

```text
host -> MCP client -> MCP server -> tool/resource/prompt -> result -> model
```

MCP standardizes how an LLM app discovers and invokes tools, resources, and
prompts. It is not an agent loop by itself, but it changes runtime architecture:
tools become protocol servers with schemas and lifecycle semantics.

Runtime primitives:

- host
- client
- server
- tool schema
- resources and prompts
- JSON-RPC transport
- lifecycle negotiation

Why it matters:

- Separates tool providers from agent hosts.
- Makes tool discovery and invocation more portable.
- Gives multi-agent systems a common context/tool substrate.

Risks:

- Tool authorization still needs host-side policy.
- A bad server can expose too much context or unsafe tools.
- Protocol plumbing can be mistaken for agent reasoning.

Sources:

- MCP specification: https://modelcontextprotocol.io/specification/2025-06-18
- MCP tools spec:
  https://modelcontextprotocol.io/specification/2025-06-18/server/tools

## 6. HITL Interrupt / Approval Gate

```text
tool proposal -> policy check -> interrupt -> approve/edit/reject -> resume
```

The runtime pauses before risky actions, stores the pending action, lets a human
approve, edit, or reject it, then resumes from the checkpoint.

Runtime primitives:

- risk policy
- pending action record
- interrupt/checkpoint
- human decision
- resume semantics
- audit trail

Why it matters:

- Required for destructive, expensive, legal, financial, or public actions.
- Turns human review into a first-class runtime branch.
- Prevents "ask in chat, hope the loop remembers" failure modes.

Risks:

- Approval fatigue.
- Bad policies either block too much or too little.
- Resume needs exact state, not just a summary.

Sources:

- LangChain HITL middleware:
  https://docs.langchain.com/oss/python/langchain/human-in-the-loop
- LangGraph overview:
  https://docs.langchain.com/oss/python/langgraph/overview

## 7. Guardrail Pipeline

```text
input -> input guardrails -> model/tool/handoff -> output guardrails -> route
```

Guardrails are runtime stages that inspect inputs, tool calls, handoffs, or
outputs and can block, transform, escalate, or continue.

Runtime primitives:

- input guardrails
- tool guardrails
- handoff policy
- output guardrails
- block/escalate/continue route
- trace event

Why it matters:

- Keeps safety and policy out of the main prompt.
- Allows different checks at different runtime boundaries.
- Makes policy failures inspectable.

Risks:

- Coverage gaps between tool and handoff paths.
- False positives can harm usability.
- Guardrails need versioning and tests.

Sources:

- OpenAI Agents SDK guardrails:
  https://openai.github.io/openai-agents-python/guardrails/
- OpenAI Agents SDK tracing:
  https://openai.github.io/openai-agents-python/tracing/

## 8. Run Receipt / Trace-First Runtime

```text
run -> record model/tool/handoff/guardrail events -> inspect/evaluate/replay
```

The runtime emits a structured record of what happened: prompts, tool calls,
handoffs, guardrail events, costs, artifacts, and final state. This becomes the
basis for debugging, evaluation, audit, and replay.

Runtime primitives:

- run ID
- trace spans/events
- artifact links
- cost/token records
- redaction policy
- replay or support bundle

Why it matters:

- Makes agent failures diagnosable.
- Enables evaluation loops over real runs.
- Supports audits and incident review.

Risks:

- Traces can leak private context.
- Redaction must happen before sharing.
- Logs without replay/checkpointing are only half useful.

Sources:

- OpenAI Agents SDK tracing:
  https://openai.github.io/openai-agents-python/tracing/
- OpenAI Agents guide:
  https://developers.openai.com/api/docs/guides/agents

## 9. Context Packing / Compression Loop

```text
context sources -> rank/filter -> pack -> budget check -> summarize/drop -> model
```

The runtime treats context as a scarce resource. It retrieves, ranks, slices,
summarizes, and drops context before model invocation, then updates summaries or
memory after the run.

Runtime primitives:

- source inventory
- relevance ranker
- context budget
- summary artifacts
- compression trigger
- memory update

Why it matters:

- Essential for long-running agents.
- Keeps prompts useful instead of merely large.
- Works with prefix caching and dynamic context injection.

Risks:

- Compression can delete decisive details.
- Recency and salience heuristics can be wrong.
- Hidden summaries need provenance.

Sources:

- DeerFlow-style durable context and dynamic context notes in this repo:
  `docs/deer-flow-techniques.md`
- OpenClaw context lifecycle notes in this repo:
  `docs/openclaw-hermes-techniques.md`

## 10. Task Queue / Resumable Workflow Runtime

```text
request -> enqueue task -> worker run -> checkpoint/status -> resume/retry/done
```

Agent work is represented as durable tasks rather than a single synchronous chat
turn. Workers can checkpoint, stream progress, retry, sleep, wait for external
events, or resume after failure.

Runtime primitives:

- task ID
- queue
- worker lease
- checkpoint
- status contract
- retry/backoff
- artifacts

Why it matters:

- Needed for long-running or background agents.
- Separates user interaction from execution lifecycle.
- Enables crash recovery and progress reporting.

Risks:

- Requires idempotency and leases.
- Partial side effects need compensation.
- Status contracts must be honest and machine-readable.

Sources:

- LangGraph durable execution overview:
  https://docs.langchain.com/oss/python/langgraph/overview
- OpenClaw heartbeat/cron/session notes in this repo:
  `docs/openclaw-hermes-techniques.md`

## Shortlist Still Worth Adding Later

- Contract-net / auction-based delegation.
- Debate with memory masking.
- Capability-aware tool reliability scoring.
- Prompt-injection defense loops.
- Runtime policy-as-code for budgets and data access.
- Artifact store plus provenance graph.
