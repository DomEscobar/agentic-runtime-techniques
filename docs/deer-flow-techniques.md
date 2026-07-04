# DeerFlow Technique Notes

Status: 2026-07-04

Source:

- GitHub: https://github.com/bytedance/deer-flow
- Local inspection clone: `/tmp/deer-flow`

DeerFlow describes itself as an open-source **super agent harness** that
orchestrates sub-agents, memory, sandboxes, tools, skills, and gateway/message
interfaces for long-horizon tasks. The repository is especially useful for this
catalog because many runtime techniques are implemented as explicit middleware.

## High-Level Runtime Shape

DeerFlow 2.0 uses a Gateway-embedded LangGraph-compatible runtime:

- LangGraph-compatible runs/threads API
- Embedded agent runtime in FastAPI Gateway
- SSE streaming
- Checkpointing
- Skills, MCP, uploads, artifacts, models
- Thread state cleanup

The graph registry points to:

```json
{
  "graphs": {
    "lead_agent": "deerflow.agents:make_lead_agent"
  }
}
```

## Core Techniques Found

| Technique | Runtime Area | Evidence | Catalog Placement |
| --- | --- | --- | --- |
| Middleware-composed agent runtime | Agent harness | `lead_agent/agent.py` builds an ordered middleware chain around `create_agent`. | Runtime Composition |
| ThreadState extension | Typed state | `ThreadState` extends `AgentState` with sandbox, thread data, title, artifacts, todos, goal, uploads, viewed images, promoted tools, delegations, skill context, summary text. | Typed State / Durable State |
| Reducer-based state merging | State correctness | Custom reducers merge sandbox, artifacts, todos, goals, promoted tools, delegations, and skill context. Conflicting sandbox IDs fail closed. | State Reducers / Conflict Handling |
| Session goals | Goal/verifier loop | `/goal` attaches a completion condition; evaluator checks visible conversation and queues hidden continuations when blocker is `goal_not_met_yet`. | Goal Loop / Verifier Loop |
| Goal continuation safety caps | Termination semantics | Defaults: max 8 continuations, max 2 no-progress continuations, typed blockers. | Termination / No-Progress Breaker |
| Plan mode todo tracking | Planning loop | `TodoMiddleware` gives the agent a `write_todos` tool with exactly-one-in-progress discipline. | Task Planning / Progress Tracking |
| Progressive skill loading | Context engineering | README says skills are loaded progressively; slash skill activation injects full `SKILL.md` only when requested. | Skill Activation / Context Budgeting |
| Slash skill activation | Explicit workflow routing | `SkillActivationMiddleware` parses `/skill-name`, validates availability, loads full `SKILL.md`, injects hidden activation context. | Intent Routing / Skill Context |
| Skill tool-policy filtering | Tool policy | Lead agent filters tools by skill allowed-tools policy before tool binding. | Tool Governance |
| Deferred tool discovery | Tool schema budgeting | `tool_search` hides MCP schemas until searched/promoted; promotions are scoped by catalog hash in state. | Deferred Tools / Tool Discovery |
| Durable context middleware | Long-context continuity | Captures delegation ledger and loaded skill files; injects summary, ledger, skills as hidden durable-context data with an authority contract. | Durable Context / Context Shaping |
| Delegation ledger | Anti-duplicate delegation | Captures `task` tool calls and paired results; renders “already delegated / do not delegate again” guidance. | Delegation Memory / Anti-Repeat |
| Subagent task tool | Multi-agent orchestration | `task` delegates to typed subagents, polls background execution, streams `task_started/running/completed/failed`, returns structured status. | Subagent Fan-Out / Status Contract |
| Subagent context isolation | Context isolation | README states each sub-agent runs in isolated context; code disables recursive subagent tools for subagents. | Isolated Subagents |
| Subagent token attribution | Cost accounting | Subagent token usage is cached/reported back to parent run journal. | Cost Attribution |
| Subagent cancellation and cleanup | Lifecycle control | Subagent executor tracks statuses, cancellation, timeout, deferred cleanup. | Lifecycle / Cancellation |
| Subagent limit middleware | Concurrency control | Lead agent installs `SubagentLimitMiddleware` when subagents are enabled. | Concurrency Limits |
| Sandbox provider abstraction | Execution isolation | Local, AIO/Docker, E2B sandbox providers; per-thread workspace/uploads/outputs paths. | Sandbox Runtime |
| Sandbox audit middleware | Runtime safety | Runtime middleware chain includes `SandboxAuditMiddleware`. | Sandbox Audit |
| Read-before-write gate | File safety | `ReadBeforeWriteMiddleware` blocks modifying existing files unless current version was read; writes invalidate earlier read marks. | Read-Before-Write / File Version Gate |
| File operation locks | Race prevention | Read-before-write gate serializes check + execution per scope/path. | Concurrency / Race Prevention |
| Guardrail middleware | Pre-tool authorization | Guardrails evaluate every tool call before execution; supports allowlist/OAP/custom providers and fail-closed behavior. | Tool Authorization / Policy Gate |
| Dangling tool-call repair | Provider compatibility | Inserts synthetic `ToolMessage`s after interrupted or malformed tool calls before next model invocation. | Tool-Call Recovery |
| Tool error handling | Failure recovery | Converts tool exceptions to error `ToolMessage`s so the run can continue; preserves LangGraph control-flow exceptions. | Tool Error Recovery |
| Safety finish reason suppression | Safety gate | If provider safety-stops with partial tool calls, strip tool calls and emit/audit safety termination. | Safety Termination / Tool Suppression |
| Loop detection middleware | Loop breaker | Hashes repeated tool call sets, warns after threshold, hard-stops by stripping tool calls. Also tracks high-frequency tools. | Loop Detection / Circuit Breaker |
| Token budget middleware | Budget enforcement | Tracks per-run input/output/total tokens, warns near threshold, hard-stops by stripping tool calls. | Token Budget / Cost Circuit Breaker |
| Tool output budget middleware | Context budgeting | Shared runtime middleware includes tool output budget control. | Tool Output Compression/Budgeting |
| Dynamic context middleware | Prompt cache + context injection | Keeps system prompt static, injects current date and memory as hidden reminders; handles date rollover. | Dynamic Context / Prefix Cache |
| Memory middleware | Long-term memory | Queues filtered user/final assistant conversation for debounced async memory update; detects corrections/reinforcement. | Memory Update Queue |
| Summarization middleware | Context compression | Configurable summarization with a pre-summarization memory flush hook. | Context Summarization |
| System message coalescing | Provider compatibility | Coalesces multiple `SystemMessage`s into one leading system message for strict providers. | Provider Compatibility |
| Input sanitization | Prompt safety | Runtime middleware includes `InputSanitizationMiddleware`. | Input Sanitization |
| Clarification middleware | Human-in-the-loop | `ask_clarification` and clarification middleware route uncertain work back to user. | HITL / Clarification Gate |
| Model factory with capability flags | Runtime model selection | Model config tracks thinking, reasoning effort, vision, Responses API, provider classes. | Model Routing / Capability Awareness |
| Tracing callbacks at graph root | Observability | Lead agent attaches tracing callbacks at invocation root to avoid duplicate spans and preserve session/user IDs. | Tracing / Observability |
| Run event stores | Audit trail | Runtime event stores include memory, JSONL, and DB implementations. | Run Journal / Auditability |
| Stream bridge | Durable streaming | Runtime has memory and Redis stream bridge implementations for SSE replay/cross-worker delivery. | Streaming / Replay |
| Support bundle with redaction | Ops/debug loop | `make support-bundle` emits issue summary/draft and redacted diagnostics, excluding `.env` and raw conversation/user file contents. | Diagnostics / Support Loop |

## DeerFlow Middleware Chain

The lead agent composes runtime behavior through middleware. The important order
is roughly:

1. Input sanitization
2. Tool output budget
3. Thread data setup
4. Upload processing
5. Sandbox acquisition
6. Dangling tool-call patch
7. LLM error handling
8. Guardrails
9. Sandbox audit
10. Read-before-write
11. Tool error handling
12. Dynamic context
13. Skill activation
14. Durable context
15. Summarization
16. Plan-mode todos
17. Token usage
18. Title generation
19. Memory queueing
20. Vision support
21. Deferred tool filtering
22. System-message coalescing
23. Subagent limits
24. Loop detection
25. Token budget
26. Safety finish-reason suppression
27. Clarification handling

This is one of the cleanest examples of an **agent runtime as middleware stack**.

## Techniques To Add To Main Taxonomy

These should become first-class taxonomy entries or subentries:

- Middleware-composed agent runtime
- Reducer-based graph state channels
- Goal evaluator with hidden continuations
- No-progress breaker
- Plan-mode todo state
- Deferred tool discovery and catalog-hash scoped promotion
- Durable delegation ledger
- Skill context capture/reinjection
- Read-before-write file version gate
- Dangling tool-call repair
- Safety finish reason tool suppression
- Tool output budget
- Token budget circuit breaker
- System-message coalescing
- Dynamic context frozen-snapshot / prefix-cache pattern
- Memory update queue with correction/reinforcement detection
- Subagent status contract
- Subagent usage attribution
- Stream bridge with replay
- Support bundle / redacted diagnostic artifact

## Tier Placement

DeerFlow itself should not be ranked as a single benchmarked technique. It is a
case-study repo for many runtime primitives. In the existing tier list it belongs
near **B Tier** as a high-signal engineering harness:

- Strong public implementation evidence
- Many tests for runtime behavior
- Broad agentic runtime coverage
- No single clean public benchmark isolating the harness as a technique

## Notes

The repo is large and active. This inspection focused on README, architecture
docs, LangGraph entrypoint, lead-agent middleware, goal runtime, subagent
executor, guardrails, memory, durable context, and defensive safety middleware.
