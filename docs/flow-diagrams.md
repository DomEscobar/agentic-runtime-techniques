# ASCII Flow Diagrams

Status: 2026-07-04

These diagrams show the control-flow shape of common agentic runtime
techniques. They are intentionally plain ASCII so they render in GitHub,
terminals, READMEs, issues, and model context without special tooling.

## 1. Core ReAct / Tool Loop

```text
User request
    |
    v
+------------------+
| Assemble context |
+------------------+
    |
    v
+------------------+
| Model reasons    |
+------------------+
    |
    v
+------------------+
| Tool call?       |
+------------------+
    | yes                         no
    v                             v
+------------------+        +------------------+
| Execute tool     |        | Final answer     |
+------------------+        +------------------+
    |
    v
+------------------+
| Observe result   |
+------------------+
    |
    +-----------------------> back to Model reasons
```

Branch notes:

- `tool call? yes` continues the loop.
- `tool call? no` terminates.
- This loop needs separate budget, verifier, or loop-detection controls in
  production.

## 2. Intent / Context / Routing Front Door

```text
Incoming request
    |
    v
+----------------------+
| Intent analysis      |
| task type, risk, SLA |
+----------------------+
    |
    v
+----------------------+
| Need more context?   |
+----------------------+
    | yes                         no
    v                             v
+----------------------+     +----------------------+
| Retrieve context     |     | Route directly       |
| files, memory, web   |     | answer/tool/workflow |
+----------------------+     +----------------------+
    |
    v
+----------------------+
| Shape/compress       |
| summaries, packets   |
+----------------------+
    |
    v
+----------------------+
| Route                |
+----------------------+
    | simple      | tool       | workflow     | ask
    v             v            v              v
 Answer      Tool loop     Program/agent   Clarify
```

Branch notes:

- Query analysis and context acquisition are runtime techniques, not just
  prompt engineering.
- Good routing prevents unnecessary agent loops.

## 3. Plan-and-Execute Loop

```text
Objective
    |
    v
+------------------+
| Build plan       |
+------------------+
    |
    v
+------------------+
| Execute step     |
+------------------+
    |
    v
+------------------+
| Observe result   |
+------------------+
    |
    v
+------------------+
| Plan still good? |
+------------------+
    | yes                         no
    v                             v
+------------------+        +------------------+
| More steps?      |        | Revise plan      |
+------------------+        +------------------+
    | yes       no                 |
    v          v                   |
 Execute   Final answer            |
 next step                         |
    ^                              |
    +------------------------------+
```

Branch notes:

- Stale plans are the main failure mode.
- A good runtime keeps the plan inspectable and mutable.

## 4. Verifier / Goal Loop

```text
Goal / completion condition
    |
    v
+------------------+
| Attempt work     |
+------------------+
    |
    v
+------------------+
| Verify outcome   |
+------------------+
    |
    v
+------------------+
| Satisfied?       |
+------------------+
    | yes                         no
    v                             v
+------------------+        +----------------------+
| Complete         |        | Blocker type?        |
+------------------+        +----------------------+
                              | fixable   | needs user | failed
                              v           v            v
                         Continue     Ask / wait    Fail / report
                              |
                              v
                         Attempt work
```

Typical typed blockers:

- `goal_not_met_yet`
- `missing_evidence`
- `needs_user_input`
- `run_failed`
- `external_wait`

## 5. Evaluator / Optimizer Loop

```text
Draft answer
    |
    v
+---------------------+
| Has rubric?         |
+---------------------+
    | yes                         no
    v                             v
+---------------------+      +---------------------+
| Trigger present?    |      | Single-pass output  |
+---------------------+      +---------------------+
    | yes        no
    v           v
+---------------------+      +---------------------+
| Separate evaluator  |      | Fast-path output    |
+---------------------+      +---------------------+
    |
    v
+---------------------+
| Passed?             |
+---------------------+
    | yes                         no
    v                             v
+---------------------+      +---------------------+
| Output              |      | Optimizer rewrite   |
+---------------------+      +---------------------+
                                  |
                                  v
                           +---------------------+
                           | Loop budget left?   |
                           +---------------------+
                                  | yes       no
                                  v          v
                             Evaluate     HITL / unsure
```

Branch notes:

- The evaluator should receive a separate prompt to reduce self-preference bias.
- The loop needs a hard cap.

## 6. Ralph / Bounded Retry Loop

```text
Task
 |
 v
+----------------------+
| Start bounded agent  |
| steps/time/budget    |
+----------------------+
 |
 v
+----------------------+
| Terminal outcome?    |
+----------------------+
 | success       | failed/block/budget
 v               v
Done       +----------------------+
           | Keep useful state?   |
           +----------------------+
               | yes        no
               v           v
        Retry with      Retry fresh
        summary/state   context
               |
               v
        Start bounded agent
```

Branch notes:

- Failure is explicit signal, not a surprise.
- Fresh context can reduce loop drift, but needs state handoff.

## 7. Research Loop

```text
Research question
    |
    v
+----------------------+
| Plan query batch     |
+----------------------+
    |
    v
+----------------------+
| Search / fetch       |
+----------------------+
    |
    v
+----------------------+
| Read / extract       |
+----------------------+
    |
    v
+----------------------+
| Synthesize evidence  |
+----------------------+
    |
    v
+----------------------+
| Gaps remain?         |
+----------------------+
    | yes                         no
    v                             v
+----------------------+     +----------------------+
| Next query batch     |     | Cited report         |
+----------------------+     +----------------------+
    |
    +------------------------> Search / fetch
```

Branch notes:

- Source quality dominates output quality.
- Evidence extraction should be explicit and URL-backed.

## 8. Experiment / Autoresearch Loop

```text
Objective + metric
    |
    v
+----------------------+
| Propose experiment   |
+----------------------+
    |
    v
+----------------------+
| Apply change         |
+----------------------+
    |
    v
+----------------------+
| Run evaluation       |
+----------------------+
    |
    v
+----------------------+
| Better?              |
+----------------------+
    | yes                         no
    v                             v
+----------------------+     +----------------------+
| Keep / commit        |     | Revert / discard     |
+----------------------+     +----------------------+
    |                             |
    v                             v
+----------------------+
| Record result        |
+----------------------+
    |
    v
+----------------------+
| Continue search?     |
+----------------------+
    | yes       no
    v          v
 Propose    Final best
 next
```

Branch notes:

- Git state, metrics, and result logs are first-class runtime state.
- Bad metrics create bad hillclimbing.

## 9. Multi-Agent Planner / Executor Harness

```text
Objective
    |
    v
+----------------------+
| Planner agent        |
+----------------------+
    |
    v
+----------------------+
| Task queue           |
+----------------------+
    |
    +-----------+-----------+
    |           |           |
    v           v           v
+--------+  +--------+  +--------+
|Worker A|  |Worker B|  |Worker C|
+--------+  +--------+  +--------+
    |           |           |
    +-----------+-----------+
                |
                v
        +------------------+
        | Review / merge   |
        +------------------+
                |
                v
        +------------------+
        | More work?       |
        +------------------+
             | yes      no
             v         v
        Task queue   Final report
```

Branch notes:

- Needs task status, dedupe, and result contracts.
- Without a delegation ledger, planners often duplicate work.

## 10. Subagent Task Tool With Status Contract

```text
Lead agent
    |
    v
+----------------------+
| task(description,    |
| prompt, subagent)    |
+----------------------+
    |
    v
+----------------------+
| Validate subagent    |
| type + permissions   |
+----------------------+
    |
    v
+----------------------+
| Start background run |
+----------------------+
    |
    v
+----------------------+
| Poll status          |
+----------------------+
    | running       | completed   | failed/cancelled/timed_out
    v               v             v
 stream update   return result   return structured error
    |
    +----------------------> Poll status
```

Branch notes:

- Good subagent systems expose `started`, `running`, `completed`, `failed`,
  `cancelled`, and `timed_out`.
- Token usage and trace IDs should flow back to the parent.

## 11. Durable Heartbeat / Checkpoint Loop

```text
Scheduler / heartbeat
    |
    v
+----------------------+
| Load state           |
+----------------------+
    |
    v
+----------------------+
| Anything useful now? |
+----------------------+
    | yes                         no
    v                             v
+----------------------+     +----------------------+
| Do work/checks       |     | Quiet checkpoint     |
+----------------------+     +----------------------+
    |
    v
+----------------------+
| Emit checkpoint      |
| progress/done/block  |
+----------------------+
    |
    v
+----------------------+
| Notify user?         |
+----------------------+
    | yes       no
    v          v
 Deliver    Stay quiet
    |
    v
+----------------------+
| Sleep / next check   |
+----------------------+
```

Branch notes:

- Heartbeats should create useful progress, not rote chatter.
- Structured checkpoint outcomes make the loop operable.

## 12. Tool Policy / Approval Gate

```text
Model tool call
    |
    v
+----------------------+
| Normalize schema     |
+----------------------+
    |
    v
+----------------------+
| Policy evaluate      |
| allow/deny/ask       |
+----------------------+
    | allow                      | deny                 | ask
    v                            v                      v
+----------------------+   +----------------------+  +----------------------+
| Execute tool         |   | Return ToolMessage   |  | Human approval       |
+----------------------+   | error to agent       |  +----------------------+
    |                     +----------------------+       | yes       no
    v                                                   v          v
+----------------------+                          Execute     Deny message
| Tool result          |
+----------------------+
```

Branch notes:

- Sandboxing is not semantic authorization.
- Policy should run before tool execution and fail closed for high-risk tools.

## 13. Deferred Tool Discovery

```text
Agent prompt
    |
    v
+-------------------------------+
| Sees deferred tool names only |
+-------------------------------+
    |
    v
+-------------------------------+
| Needs hidden tool?            |
+-------------------------------+
    | yes                         no
    v                             v
+-------------------------------+  Use active tools
| tool_search(query)            |
+-------------------------------+
    |
    v
+-------------------------------+
| Promote matched schemas       |
| scoped by catalog hash        |
+-------------------------------+
    |
    v
+-------------------------------+
| Tool becomes callable         |
+-------------------------------+
```

Branch notes:

- Keeps huge MCP catalogs out of the model context.
- Catalog-hash scoping prevents stale tool names from exposing drifted tools.

## 14. Read-Before-Write File Gate

```text
write_file / str_replace
    |
    v
+-------------------------------+
| File already exists?          |
+-------------------------------+
    | no                          yes
    v                             v
 Allow create             +-------------------------------+
                          | Has current read mark?        |
                          +-------------------------------+
                              | yes                    no
                              v                       v
                       Allow write        Block with ToolMessage error
                              |
                              v
                       Invalidate old marks
```

Branch notes:

- Prevents append-only duplication and stale edits.
- The read mark should be tied to file content hash, not just path.

## 15. Dangling Tool-Call Repair

```text
Before model call
    |
    v
+-------------------------------+
| Scan history for AI tool_calls|
+-------------------------------+
    |
    v
+-------------------------------+
| Matching ToolMessages exist?  |
+-------------------------------+
    | yes                         no
    v                             v
 Leave history            +-------------------------------+
 unchanged                | Insert synthetic error        |
                          | ToolMessage immediately after |
                          | dangling AIMessage            |
                          +-------------------------------+
                              |
                              v
                       Send well-formed history
```

Branch notes:

- Needed for strict providers that validate tool-call IDs.
- Useful after user interruption, cancellation, malformed tool args, or provider
  partial output.

## 16. Loop Detection Circuit Breaker

```text
After model response
    |
    v
+-------------------------------+
| Hash tool calls + salient args|
+-------------------------------+
    |
    v
+-------------------------------+
| Same call repeated?           |
+-------------------------------+
    | below warn   | warn threshold       | hard limit
    v              v                      v
 Continue     Queue warning          Strip tool_calls
              for next model call    force final answer
```

Branch notes:

- Warning injection should happen at the next model call, after prior
  ToolMessages exist, to preserve tool-call pairing.
- Track both exact repeats and high-frequency tool types.

## 17. Token Budget Circuit Breaker

```text
After model response
    |
    v
+-------------------------------+
| Sum run token usage           |
| input/output/total            |
+-------------------------------+
    |
    v
+-------------------------------+
| Budget fraction?              |
+-------------------------------+
    | under warn   | near limit          | hard stop
    v              v                     v
 Continue     Inject warning       Strip tool_calls
              next turn            produce final answer
```

Branch notes:

- Subagent token use should be attributed back to the parent run.
- Hard stop should preserve collected results.

## 18. Context / Memory Update Loop

```text
Run finishes
    |
    v
+------------------------------+
| Filter messages              |
| user + final assistant only  |
+------------------------------+
    |
    v
+------------------------------+
| Correction or reinforcement? |
+------------------------------+
    | correction     | reinforcement    | neutral
    v                v                  v
 Mark fix       Mark preference      Normal memory
    |                |                  |
    +----------------+------------------+
                     |
                     v
              +------------------+
              | Debounced queue  |
              +------------------+
                     |
                     v
              +------------------+
              | Async summarize  |
              | and persist      |
              +------------------+
```

Branch notes:

- Memory updates should ignore raw tool chatter.
- Debouncing prevents excessive memory writes.

## 19. Coding Harness Loop

```text
Task
 |
 v
+----------------------+
| Create branch/workdir|
+----------------------+
 |
 v
+----------------------+
| Implement            |
+----------------------+
 |
 v
+----------------------+
| Run checks/tests     |
+----------------------+
 |
 v
+----------------------+
| Review               |
+----------------------+
 | pass                       | fail
 v                            v
Merge / handoff        Fix or rollback
                              |
                              v
                       Run checks/tests
```

Branch notes:

- Strong versions add read-before-write gates, test evidence, run receipts,
  rollback, and independent review.

## 20. DeerFlow-Style Middleware Stack

```text
User message
    |
    v
+--------------------------+
| Input sanitization       |
+--------------------------+
    |
    v
+--------------------------+
| Thread data + uploads    |
+--------------------------+
    |
    v
+--------------------------+
| Sandbox acquisition      |
+--------------------------+
    |
    v
+--------------------------+
| Tool/runtime guards      |
| dangling calls, errors,  |
| guardrails, read-before- |
| write, sandbox audit     |
+--------------------------+
    |
    v
+--------------------------+
| Context shaping          |
| dynamic context, skills, |
| durable context, summary |
+--------------------------+
    |
    v
+--------------------------+
| Planning / todos         |
+--------------------------+
    |
    v
+--------------------------+
| Agent model + tools      |
+--------------------------+
    |
    v
+--------------------------+
| Post-model controls      |
| loop detect, token budget|
| safety finish suppression|
+--------------------------+
    |
    v
+--------------------------+
| Clarification or answer  |
+--------------------------+
```

Branch notes:

- This is a runtime stack, not a single loop.
- Middleware order matters because some layers repair history, some gate tools,
  and some modify model-bound context.

## 21. OpenClaw-Style Operational Runtime

```text
Inbound event
    |
    v
+--------------------------+
| Delivery metadata        |
| channel/account/thread   |
+--------------------------+
    |
    v
+--------------------------+
| Bootstrap context        |
| AGENTS/SOUL/TOOLS/etc.   |
+--------------------------+
    |
    v
+--------------------------+
| Agent turn               |
| tools, memory, sessions  |
+--------------------------+
    |
    v
+--------------------------+
| Need delegation?         |
+--------------------------+
    | yes                         no
    v                             v
+--------------------------+  Continue same turn
| sessions_spawn / ACP     |
| child agent              |
+--------------------------+
    |
    v
+--------------------------+
| Delivery decision        |
| final private? visible?  |
+--------------------------+
    |
    v
+--------------------------+
| Send / queue / stay quiet|
+--------------------------+
```

Operational branches:

```text
Heartbeat
   |
   v
Read HEARTBEAT.md
   |
   v
Useful action now?
   | yes                         no
   v                             v
Do background work          HEARTBEAT_OK / quiet
   |
   v
Notify only if worth interrupting
```

## 22. Onyx / Programmed Agent Runtime

```text
*.program.ts
    |
    v
+------------------------+
| Deterministic code     |
| owns control flow      |
+------------------------+
    |
    v
+------------------------+
| run(agent) or spawn()  |
+------------------------+
    | run/blocking              | spawn/background
    v                           v
Await result              Get handle
    |                           |
    v                           v
+------------------------+  +------------------------+
| Typed state update     |  | steer / cancel / wait  |
+------------------------+  +------------------------+
    |
    v
+------------------------+
| sleep/checkpoint?      |
+------------------------+
    | yes                         no
    v                             v
Long-running loop       Continue program
```

Branch notes:

- The runtime treats agent failure as code-level error semantics.
- `sleep` + `checkpoint` turns a program into a durable agent loop.

## 23. Supervisor Architecture

```text
User request
    |
    v
+------------------------+
| Supervisor agent       |
| route / delegate       |
+------------------------+
    |
    +------------+------------+
    |            |            |
    v            v            v
+---------+  +---------+  +---------+
|Agent A  |  |Agent B  |  |Agent C  |
+---------+  +---------+  +---------+
    |            |            |
    +------------+------------+
                 |
                 v
        +------------------------+
        | Back to supervisor     |
        | synthesize / forward   |
        +------------------------+
                 |
                 v
        +------------------------+
        | More delegation?       |
        +------------------------+
             | yes       no
             v          v
        Supervisor   User response
```

Branch notes:

- Only the supervisor talks to the user in the strict form.
- Subagents can be treated as black boxes, which makes this generic.
- The main risk is the "telephone" problem: supervisor paraphrases or distorts
  subagent outputs.
- Strong versions add `forward_message`, handoff-message trimming, typed
  blockers, and delegation ledgers.

## 24. Swarm Architecture

```text
User request
    |
    v
+------------------------+
| Active agent           |
+------------------------+
    |
    v
+------------------------+
| Can handle?            |
+------------------------+
    | yes                         no / better specialist
    v                             v
+------------------------+   +------------------------+
| Respond / act          |   | Handoff tool           |
+------------------------+   | transfer to agent X    |
    |                       +------------------------+
    v                             |
+------------------------+        v
| Done?                  |   +------------------------+
+------------------------+   | New active agent       |
    | yes       no          +------------------------+
    v          v                    |
  User     Active agent <-----------+
 response
```

Branch notes:

- Agents hand off directly to one another.
- One agent is active at a time in the common LangGraph/OpenAI Swarm shape.
- The system should remember the last active agent for later turns.
- The main risk is loss of global coordination: no central supervisor owns the
  whole plan unless you add one.

## 25. Hierarchical Supervisor Tree

```text
User request
    |
    v
+--------------------------+
| Root supervisor          |
+--------------------------+
    |
    +------------------+------------------+
    |                                     |
    v                                     v
+--------------------------+      +--------------------------+
| Domain supervisor A      |      | Domain supervisor B      |
+--------------------------+      +--------------------------+
    |                                     |
    +-----------+-----------+             +-----------+-----------+
    |           |           |             |           |           |
    v           v           v             v           v           v
 Agent A1    Agent A2    Agent A3      Agent B1    Agent B2    Agent B3
    |           |           |             |           |           |
    +-----------+-----------+             +-----------+-----------+
                |                                     |
                v                                     v
        Domain supervisor A                  Domain supervisor B
                |                                     |
                +------------------+------------------+
                                   |
                                   v
                           Root supervisor
                                   |
                                   v
                            User response
```

Branch notes:

- Useful when domains are too large for one supervisor.
- Adds coordination cost and more places for information loss.
- Needs explicit ownership, delegation limits, and result contracts.

## 26. Blackboard / Shared Workspace Coordination

```text
Goal / problem
    |
    v
+--------------------------+
| Shared blackboard        |
| tasks, claims, evidence  |
+--------------------------+
    |
    +-------------+--------------+-------------+
    |             |              |             |
    v             v              v             v
+---------+   +---------+    +---------+   +---------+
|Agent A  |   |Agent B  |    |Agent C  |   |Agent D  |
+---------+   +---------+    +---------+   +---------+
    |             |              |             |
    +-------------+--------------+-------------+
                  |
                  v
          +--------------------------+
          | Post updates / conflicts |
          +--------------------------+
                  |
                  v
          +--------------------------+
          | Arbiter / synthesizer    |
          +--------------------------+
              | enough        | gaps/conflicts
              v               v
          Final answer     Back to blackboard
```

Branch notes:

- Agents coordinate through shared state, not direct chat.
- The blackboard needs freshness and ownership rules.

## 27. Multi-Agent Debate / Committee Deliberation

```text
Question
   |
   v
+--------------------------+
| Independent proposals    |
+--------------------------+
   |
   +-----------+-----------+
   |           |           |
   v           v           v
Agent A     Agent B     Agent C
   |           |           |
   +-----------+-----------+
               |
               v
      +--------------------------+
      | Debate / critique round  |
      +--------------------------+
               |
               v
      +--------------------------+
      | Stable or budget spent?  |
      +--------------------------+
          | yes             no
          v                v
   Judge / vote /       Next debate
   aggregate            round
          |
          v
       Final
```

Branch notes:

- Debate is useful only when diversity and correction beat added cost.
- Needs a round budget and a final aggregation rule.

## 28. Agent-as-Judge / Referee Team

```text
Candidate artifact
    |
    v
+--------------------------+
| Rubric / acceptance spec |
+--------------------------+
    |
    v
+--------------------------+
| Judge or referee panel   |
+--------------------------+
    |
    v
+--------------------------+
| Decision                 |
+--------------------------+
  | accept      | revise       | reject/escalate
  v             v              v
Output      Back to worker   Human / failure
```

Branch notes:

- This is a runtime gate, not just offline eval.
- Strong versions keep score schemas and rationale separate from the final
  artifact.

## 29. A2A / Agent Interoperability Protocol Layer

```text
Local agent
    |
    v
+--------------------------+
| Discover agent card      |
+--------------------------+
    |
    v
+--------------------------+
| Can remote agent help?   |
+--------------------------+
    | yes                         no
    v                             v
+--------------------------+   Use local route
| Send / stream task       |
+--------------------------+
    |
    v
+--------------------------+
| Status / artifact events |
+--------------------------+
    |
    v
+--------------------------+
| Done?                    |
+--------------------------+
    | yes          no
    v             v
 Use result    Poll / stream
```

Branch notes:

- Remote agents become task endpoints with capability manifests.
- Trust, auth, status semantics, and artifacts matter as much as prompts.

## 30. MCP Tool / Context Protocol Layer

```text
LLM host
   |
   v
+--------------------------+
| MCP client               |
+--------------------------+
   |
   v
+--------------------------+
| Server capability list   |
| tools/resources/prompts  |
+--------------------------+
   |
   v
+--------------------------+
| Invoke tool/resource?    |
+--------------------------+
   | yes                         no
   v                             v
+--------------------------+   Continue model
| MCP server executes      |
+--------------------------+
   |
   v
+--------------------------+
| Result to host/model     |
+--------------------------+
```

Branch notes:

- MCP is a protocol substrate, not an agent brain.
- Host-side authorization and context minimization still matter.

## 31. HITL Interrupt / Approval Gate

```text
Proposed action
    |
    v
+--------------------------+
| Policy check             |
+--------------------------+
    |
    v
+--------------------------+
| Human review required?   |
+--------------------------+
    | yes                         no
    v                             v
+--------------------------+   Execute action
| Interrupt + checkpoint   |
+--------------------------+
    |
    v
+--------------------------+
| Human decision           |
+--------------------------+
  | approve    | edit         | reject
  v            v              v
Execute     Execute edited   Resume alternate path
```

Branch notes:

- The pending action needs exact arguments, not a vague summary.
- Resume semantics are part of the pattern.

## 32. Guardrail Pipeline

```text
Input
  |
  v
+--------------------------+
| Input guardrails         |
+--------------------------+
  | pass        | block/escalate
  v             v
Model run     Stop / human
  |
  v
+--------------------------+
| Tool or handoff proposed |
+--------------------------+
  |
  v
+--------------------------+
| Boundary-specific policy |
+--------------------------+
  | pass        | block/escalate
  v             v
Execute       Stop / human
  |
  v
+--------------------------+
| Output guardrails        |
+--------------------------+
  | pass        | revise/block
  v             v
Final        Back to model / human
```

Branch notes:

- Different boundaries need different checks.
- Guardrail events should be traced.

## 33. Task Queue / Resumable Workflow Runtime

```text
Request
   |
   v
+--------------------------+
| Enqueue task             |
+--------------------------+
   |
   v
+--------------------------+
| Worker leases task       |
+--------------------------+
   |
   v
+--------------------------+
| Run step                 |
+--------------------------+
   |
   v
+--------------------------+
| Checkpoint / status      |
+--------------------------+
   |
   v
+--------------------------+
| Terminal?                |
+--------------------------+
   | done       | wait/sleep   | fail/retry
   v            v             v
Complete    Resume later   Backoff / new worker
```

Branch notes:

- This turns agent work into durable jobs instead of chat turns.
- Requires idempotency, leases, and honest status contracts.

## 34. Stateful Harness Around Pure Agent Loop

```text
Frontend / session layer
    |
    v
+--------------------------+
| AgentHarness             |
| transcript, queues,      |
| listeners, cancellation  |
+--------------------------+
    |
    v
+--------------------------+
| run_agent_loop           |
| provider + tools         |
+--------------------------+
    |
    v
+--------------------------+
| Provider-neutral events  |
+--------------------------+
    |
    v
Frontend renders / persists
```

Interactive branches:

```text
Run active?
   | no                          yes
   v                             v
prompt/continue             queue user input
                                  |
                                  v
                          steering or follow-up?
                             | steering      | follow-up
                             v               v
                    inject after safe    inject when agent
                    turn/tool boundary   would otherwise stop
```

Cancellation/repair branch:

```text
cancel run
   |
   v
assistant tool calls missing results?
   | no                          yes
   v                             v
resume normally             append failed tool results
                                  |
                                  v
                            next provider request is valid
```

Branch notes:

- The pure loop stays stateless; the harness owns transcript and runtime state.
- Queues preserve a single-runner invariant while still allowing interactive
  steering.
- Interrupted tool-call repair keeps OpenAI-compatible transcripts valid after
  cancellation.
