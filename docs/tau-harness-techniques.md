# Tau Harness Technique Notes

Status: 2026-07-04

Source:

- https://github.com/huggingface/tau/blob/main/src/tau_agent/harness.py
- https://github.com/huggingface/tau/blob/main/src/tau_agent/loop.py
- https://github.com/huggingface/tau/blob/main/src/tau_agent/events.py
- https://github.com/huggingface/tau/blob/main/src/tau_agent/tools.py

Tau is useful for this repo because `tau_agent/harness.py` is a small, readable
example of a stateful reusable agent brain wrapped around a stateless/pure
provider-tool loop. The interesting part is not that it is a coding agent; it is
the boundary between loop, harness, UI/session layer, event stream, queued user
steering, cancellation, and transcript repair.

## Core Pattern: Stateful Harness Around Pure Loop

```text
frontend/session
    |
    v
+--------------------------+
| AgentHarness             |
| owns transcript + queues |
+--------------------------+
    |
    v
+--------------------------+
| run_agent_loop           |
| pure provider/tool loop  |
+--------------------------+
    |
    v
+--------------------------+
| provider-neutral events  |
+--------------------------+
    |
    v
frontend/session renders or persists
```

Tau separates these responsibilities:

- `run_agent_loop` is the reusable execution loop. It receives a transcript list
  owned by the caller, appends assistant/tool messages, executes tools, and
  emits typed events.
- `AgentHarness` owns mutable state: transcript, listeners, running flag,
  cancellation signal, steering queue, and follow-up queue.
- `CodingSession` and the TUI sit outside the harness. They own coding tools,
  durable session entries, rendering, commands, provider config, resources, and
  UI behavior.

This is a strong runtime design because the core loop stays portable while the
harness gives applications a stable "agent brain" object.

## Techniques To Extract

### 1. Pure Loop With Caller-Owned Transcript

`run_agent_loop` takes `messages: list[AgentMessage]` and mutates it as the run
progresses, but the loop itself does not own session persistence or UI state.

Runtime primitives:

- caller-owned transcript
- provider/model/system/tools passed as parameters
- assistant messages appended on provider response end
- tool result messages appended after tool execution
- `max_turns` bound
- cancellation signal

Why it matters:

- The same loop can be wrapped by CLI, TUI, tests, sessions, or another agent.
- The persistence policy can change without rewriting the loop.
- The loop can be reasoned about independently from app concerns.

Failure modes covered:

- unbounded turn loops via `max_turns`
- provider stream without assistant message
- unknown tools converted into tool result messages
- tool exceptions converted into structured failed tool results

### 2. Provider-Neutral Event Stream

Tau normalizes provider events into portable agent events:

- `agent_start` / `agent_end`
- `turn_start` / `turn_end`
- `message_start` / `message_delta` / `thinking_delta` / `message_end`
- `tool_execution_start` / `tool_execution_end`
- `retry`
- `queue_update`
- `error`

Runtime primitives:

- typed event models
- event listener subscription
- async iterator event stream
- frontend-independent rendering contract

Why it matters:

- Frontends do not need to understand provider-specific streaming details.
- Sessions, renderers, loggers, and tests can consume the same event stream.
- It creates a natural run-receipt/tracing boundary.

### 3. Steering Queue

Tau supports a steering queue for user messages that should be injected after
the current turn/tool batch while the agent is still running.

```text
agent running
    |
    v
user sends steering
    |
    v
queued steering message
    |
    v
drain after tool batch / turn
    |
    v
continue loop with injected user message
```

Runtime primitives:

- steering queue
- `queue_update` event
- queue drain callback passed into the pure loop
- queue mode: `one_at_a_time` or `all`

Why it matters:

- The UI can influence an active run without violating the single-running-loop
  invariant.
- Steering is safer than mutating the transcript directly from the frontend.
- Queue updates are visible to the UI and can be rendered or edited.

### 4. Follow-Up Queue

Tau also supports follow-up messages that are injected when the agent would
otherwise stop.

```text
agent produces final assistant message
    |
    v
no tool calls
    |
    v
follow-up queue non-empty?
    | yes                         no
    v                             v
append queued user message      stop
    |
    v
continue next turn
```

Runtime primitives:

- follow-up queue
- pop latest follow-up
- clear queues
- queue update events

Why it matters:

- Users can line up the next instruction without racing the active run.
- A frontend can treat "steer now" and "ask next" as different commands.
- This is a small but important UX/runtime distinction for interactive agents.

### 5. Single-Runner Invariant

`AgentHarness` rejects `prompt()` or `continue_()` while a run is active and
directs callers to use `steer()` or `follow_up()`.

Runtime primitives:

- `is_running`
- `_ensure_not_running`
- queued messages for concurrent user input

Why it matters:

- Prevents two loops from mutating the same transcript at once.
- Makes concurrency explicit instead of accidental.
- Gives UI code a safe fallback path.

### 6. Cancellation Token

The harness creates a cancellation token for each run and passes it to the loop
and tools. Tools can observe cancellation through the same minimal interface.

Runtime primitives:

- run-scoped cancellation signal
- tool cancellation signal
- recoverable cancellation errors
- cancelled tool results for not-yet-executed tool calls

Why it matters:

- Cancellation is a normal runtime branch, not a process kill.
- The transcript can remain provider-valid after cancellation.
- Tool execution can cooperate instead of being abruptly abandoned.

### 7. Interrupted Tool-Call Repair

If a run is cancelled after an assistant message with tool calls but before all
matching tool results are appended, OpenAI-compatible providers can reject the
next request. Tau repairs the transcript by appending failed `ToolResultMessage`
entries for missing tool call IDs.

```text
previous transcript
    |
    v
latest assistant has tool_calls?
    | no                          yes
    v                             v
continue                     collect returned IDs
                                  |
                                  v
                         missing tool result?
                              | yes
                              v
                       append failed result:
                       "Tool call interrupted by user"
```

Runtime primitives:

- scan latest open assistant tool-call message
- collect returned tool call IDs
- append synthetic failed tool result for gaps
- run repair before `prompt()` and `continue_()`
- run repair after cancellation

Why it matters:

- Keeps transcript schema valid across cancellations and UI interrupts.
- Turns partial side effects into explicit failed observations.
- Same idea appears in production systems as dangling-tool-call repair.

### 8. Provider-Neutral Tool Boundary

Tau tools are schemas plus async executors returning structured results. Unknown
tools and thrown exceptions become failed tool result messages.

Runtime primitives:

- `AgentTool`
- input schema
- async executor
- `AgentToolResult`
- `ok`, `content`, `data`, `details`, `error`

Why it matters:

- Tool failures stay inside the agent loop as observations.
- Provider adapters can stay separate from tool implementation.
- Tool results become traceable artifacts, not arbitrary strings.

## Decision Guidance

Use this pattern when:

- You want one reusable agent brain that can power CLI, TUI, API, tests, or
  sessions.
- The UI needs to steer an active run without starting a second loop.
- You need provider-neutral event rendering and logging.
- Cancellation must preserve transcript validity.
- You want a small core loop with session/UI concerns outside it.

Avoid this pattern when:

- The agent is a one-shot stateless API call.
- There is no interactive steering, persistence, or cancellation requirement.
- The environment can tolerate losing the transcript after interruption.

## Relation To Existing Repo Patterns

Tau reinforces and sharpens these existing patterns:

- **Core ReAct / Tool Loop**: `run_agent_loop` is a compact provider/tool loop.
- **Run Receipt / Trace-First Runtime**: typed events form the trace boundary.
- **Task Queue / Resumable Workflow Runtime**: queues show how user input can
  enter a running loop safely.
- **Dangling Tool-Call Repair**: interrupted tool calls are repaired before the
  next provider request.
- **Context / Memory Update Loop**: caller-owned transcript enables session
  compaction and durable reconstruction outside the pure loop.

## Pattern Name

Suggested catalog name:

**Stateful Harness Around Pure Agent Loop**

Short definition:

> A reusable harness owns transcript, queueing, cancellation, listeners, and
> repair semantics while delegating model/tool execution to a stateless loop
> that emits provider-neutral events.
