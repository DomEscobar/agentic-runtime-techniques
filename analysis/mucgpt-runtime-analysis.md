# MUCGPT Agentic Runtime Analysis

Status: 2026-07-04

Repository:

- https://github.com/it-at-m/mucgpt

Analyzed revision:

- `35a28ff` (`2026-07-03T13:50:17+02:00`)
- Commit subject: `v0.27.0`

## Verdict

**Agentic runtime rating: 6.5/10 — B-/B**

MUCGPT is a solid assistant-first public-administration AI platform. As an
agentic runtime, it is strongest in tool integration, MCP handling,
assistant configuration, organization-aware access, and observability. It is
weaker in durable execution, explicit planning, verifier/goal loops,
human-in-the-loop approvals, runtime budget enforcement, and multi-agent
orchestration.

Short version:

> MUCGPT is not an autonomous deep-agent runtime. It is a productized ReAct +
> MCP assistant platform with good operational scaffolding and several useful
> runtime primitives.

## Scorecard

| Area | Score | Notes |
| --- | ---: | --- |
| Core ReAct/tool loop | 7/10 | Real LangChain/LangGraph agent loop with tools and middleware. |
| Tool/MCP runtime | 8/10 | Strong MCP loader, cache, lock, token forwarding, metadata enrichment, partial failure handling. |
| Assistant/config/capability layer | 7.5/10 | Assistants are versioned, own tools/prompts/settings, and can be shared/scoped. |
| Observability/tracing | 7/10 | Langfuse integration, hashed user IDs, assistant tags, internal stream filtering. |
| Guardrails/security runtime | 5.5/10 | Good untrusted document prompt guard, but no action firewall or HITL approval gate. |
| Durable execution/session runtime | 3/10 | Chat history appears browser-local; no server-side LangGraph checkpointer found. |
| Planning/verifier loops | 4/10 | Reflective loop exists inside Simplify tool, but main agent lacks explicit planner/verifier. |
| Multi-agent orchestration | 1/10 | No supervisor, swarm, blackboard, or multi-agent runtime found. |
| Product/platform architecture | 7.5/10 | Microservices, gateway, SSO, DB, Redis, assistant service, CI, containers. |

## What MUCGPT Does Well

### 1. ReAct Agent With Middleware

Evidence:

- `mucgpt-core-service/app/agent/react_agent.py`
- `mucgpt-core-service/app/agent/middleware.py`

MUCGPT uses LangChain's `create_agent(...)` with tools, state schema, context
schema, and middleware. This is a real tool-using ReAct-style agent, not just a
plain chat completion wrapper.

Important runtime primitives:

- LangChain agent creation.
- Request-specific tool selection.
- Context middleware.
- Tool error middleware.
- State schema selection.
- Streaming and non-streaming execution paths.

### 2. Strong MCP Tool Runtime

Evidence:

- `mucgpt-core-service/app/agent/tools/mcp.py`
- `mucgpt-core-service/app/config/settings.py`
- `docs/GETTING_STARTED.md`

The MCP layer is one of the strongest runtime areas in MUCGPT.

Notable techniques:

- Multi-server MCP client.
- SSE and Streamable HTTP transports.
- Per-user MCP tool cache in Redis.
- Redis lock to avoid cache stampede.
- Cache TTL configuration.
- Force reload.
- Partial failure handling:
  - If some MCP sources fail but others succeed, it caches partial results for a
    short TTL.
  - If all sources fail, it returns an empty set without caching the failure for
    the full TTL.
- Waits briefly for cache warm-up if another worker holds the lock.
- Falls back to uncached load if cache is still not ready.
- Per-user bearer token forwarding.
- Optional auth override.
- Custom MCP tool description overrides.
- MCP source/group metadata.

This maps well to the repo's patterns:

- MCP Tool/Context Protocol Layer.
- Tool Reliability Scoring and Fallback, partially.
- Capability / Least-Privilege Runtime, partially.
- Runtime cache and lock primitive.

### 3. Assistant Config As Capability Surface

Evidence:

- `mucgpt-assistant-service/app/database/database_models.py`
- `mucgpt-assistant-service/app/database/assistant_repo.py`
- `mucgpt-assistant-service/app/api/routers/assistants_router.py`
- `docs/FEATURES.md`

MUCGPT is assistant-first. An assistant is a reusable configuration with system
prompt, model settings, examples, quick prompts, tools, ownership, visibility,
hierarchical access, and versioning.

Runtime value:

- Assistant config constrains which tools are enabled.
- Assistants provide reusable task-specific context.
- Sharing and ownership make the system organizational, not just individual.
- Versioned assistant configs are useful for audit and rollback.

This is not a runtime loop by itself, but it is an important capability and
governance layer.

### 4. Policy Middleware and State Schema Selection

Evidence:

- `mucgpt-core-service/app/agent/tools/tools.py`
- `mucgpt-core-service/app/agent/state_models/default_state.py`
- `mucgpt-core-service/app/agent/state_models/atlassian_state.py`
- `mucgpt-core-service/app/agent/tools/policies.py`

MUCGPT selects an agent state schema based on enabled tool groups. If all
selected tools belong to one MCP group, the corresponding state schema can be
used. The strongest concrete example is `AtlassianAgentState`.

This is a useful pattern:

```text
enabled tools -> tool group metadata -> state schema -> policy -> tool filtering
```

However, see the limitation below: dynamic scope inference appears to be
present but not active in the current middleware path.

### 5. Context Injection Guard For Uploaded Data Sources

Evidence:

- `mucgpt-core-service/app/agent/middleware.py`

Uploaded documents are injected as a guarded `<data-sources>` block. The guard
explicitly tells the model that document text is untrusted reference data and
must not override system prompts or tool policies.

This is good practice and maps to:

- Context Trust Zones.
- Prompt-Injection Defense, partially.
- Source attribution / provenance, partially.

Important limitation:

- This is mostly a prompt-level context guard. It does not appear to enforce an
  action firewall that screens proposed tool calls against original user intent
  and trust labels.

### 6. Langfuse Observability

Evidence:

- `mucgpt-core-service/app/config/langfuse_provider.py`
- `mucgpt-core-service/app/agent/agent_executor.py`
- `mucgpt-core-service/app/agent/middleware.py`

MUCGPT integrates Langfuse through callbacks and decorators.

Runtime primitives:

- `@observe` spans for agent/completion runs.
- Hashed user ID propagation.
- Assistant tags.
- Internal run metadata.
- Agent state schema metadata.
- Stream filtering for internal helper/router chunks.

This maps to:

- Run Receipt / Trace-First Runtime.
- Observability / audit.

### 7. Reflective Tool-Local Evaluator Loop

Evidence:

- `mucgpt-core-service/app/agent/tools/simplify_agent.py`

The `SimplifyAgent` is a small LangGraph:

```text
generate -> critique -> refine -> critique -> ... -> end
```

It has:

- Typed state.
- Generate node.
- Critique node.
- Refine node.
- Conditional edge.
- Maximum revision count (`MAX_REVISIONS = 5`).
- Tool progress streaming.

This is a real evaluator/optimizer loop, but it is scoped to one tool. It does
not make the whole MUCGPT agent a verifier-driven runtime.

### 8. Tool Progress Streaming

Evidence:

- `mucgpt-core-service/app/agent/tools/tool_chunk.py`
- `mucgpt-core-service/app/agent/agent_executor.py`
- `mucgpt-frontend/src/utils/ToolStreamHandler.ts`

Tools can stream custom chunks, and the backend converts them to
OpenAI-compatible `tool_calls` chunks for the frontend.

Runtime value:

- Users see tool progress.
- Tools can expose intermediate stages.
- Internal helper chunks are filtered from normal assistant output.

## Main Weaknesses

### 1. No Server-Side Durable Agent State Found

Evidence:

- `docs/FEATURES.md` says chat history stays local in the browser
  (`IndexedDB`).
- No obvious LangGraph checkpointer, `thread_id`, `MemorySaver`,
  `PostgresSaver`, or persistent server-side agent state was found.

Impact:

- The backend agent is mostly request-scoped.
- Durable execution and resume are weak.
- Policy state must be inferred from request/history rather than persisted as
  runtime state.

Rating impact:

- Durable execution/session runtime: **3/10**.

### 2. Scope Router Exists But Appears Disabled

Evidence:

- `mucgpt-core-service/app/agent/middleware.py`
- `mucgpt-core-service/app/agent/tools/policies.py`

The `AtlassianScopePolicy` includes `ainfer_scope(...)`, structured
`ScopeDecision`, confidence scoring, and internal router tracing. But in
`ContextMiddleware`, the calls to `policy.infer_scope(...)` and
`policy.ainfer_scope(...)` are commented out.

The executor initializes state with:

```text
agent_state: {"current_scope": "general"}
```

Practical effect:

- The scope policy is architecturally interesting, but dynamic scope routing
  does not appear active in the current request path.
- Tool filtering may often remain in general mode unless the frontend/state
  supplies a scope.

This is a strong "almost there" runtime feature.

### 3. No Explicit Planner / Goal / Verifier For The Main Agent

The main agent is ReAct-style tool use. I did not find:

- A durable plan artifact.
- Plan-and-execute loop.
- Goal verifier.
- Typed blocker states.
- Main-agent done checker.
- Separate reviewer/evaluator gate for ordinary responses.

The `SimplifyAgent` has a verifier-like loop, but only inside one tool.

### 4. No Human-In-The-Loop Approval Gate Found

This matters because MUCGPT supports MCP and token forwarding. If MCP tools can
perform sensitive actions, the runtime should ideally include:

- Proposed action records.
- Risk classification.
- Approve/edit/reject path.
- Resume semantics.
- Audit event.

I did not find this in the current core runtime.

### 5. Runtime Budget Enforcement Is Weak

Evidence:

- Model metadata includes `max_input_tokens` and `max_output_tokens`.
- API models include usage fields.
- Non-streaming response currently returns usage as `0/0/0`.
- No obvious runtime policy for:
  - max agent turns,
  - max tool calls,
  - max delegation depth,
  - cost budget,
  - token warning threshold,
  - hard-stop/checkpoint behavior.

This is an important gap for autonomous tool use.

### 6. Prompt Injection Defense Is Partial

Strength:

- Uploaded document content is explicitly marked untrusted.

Gap:

- No action firewall found.
- No trust-zone metadata flowing into tool-call authorization.
- No screening of proposed tool calls against original user intent.

For an MCP-enabled system, this distinction matters.

### 7. Tool Reliability Is Partial

Strength:

- MCP loader handles partial source failures well.
- It avoids caching all-source failures.
- It has short-TTL partial caching.

Gap:

- No long-term tool health scores.
- No quarantine for repeatedly failing tools.
- No fallback ranking by observed reliability.
- No schema-drift score beyond local error handling.

### 8. No Multi-Agent Runtime

No evidence found for:

- Supervisor.
- Swarm.
- Blackboard.
- Multi-agent debate.
- Worker registry.
- Delegation ledger.

This is not necessarily a product problem, but it lowers the agentic-runtime
score.

## Mapping To This Repo's Runtime Patterns

| Pattern | MUCGPT Evidence | Rating |
| --- | --- | --- |
| Core ReAct / Tool Loop | LangChain `create_agent` with tools and middleware | Strong |
| MCP Tool/Context Protocol Layer | MultiServerMCPClient, per-user cache, token forwarding | Strong |
| Tool Error Recovery | `ToolErrorMiddleware` converts exceptions to tool messages | Good |
| Context Trust Zones | Data-source guard says documents are untrusted reference data | Partial |
| Prompt-Injection Action Firewall | No action-level guard found | Weak |
| Capability / Least-Privilege Runtime | Assistant-enabled tools and MCP forward-token config | Partial |
| Run Receipt / Trace-First Runtime | Langfuse, hashed user, assistant tags, metadata | Good |
| Runtime Budget Policy Engine | Mostly missing | Weak |
| Tool Reliability Scoring | Partial MCP failure handling, no scoring | Partial |
| Durable Session Event Log | Browser-local chat history, no server checkpointer found | Weak |
| Evaluator / Optimizer Loop | Present in Simplify tool only | Partial |
| Planner / Executor Harness | Not found | Missing |
| Supervisor / Swarm / Blackboard | Not found | Missing |

## Tier Placement

Suggested tier:

**B-Tier implementation evidence**

Reason:

- Strong enough to show how a real public-sector assistant platform combines
  ReAct, MCP, assistant config, organization access, streaming, and Langfuse.
- Not strong enough for A-Tier agent runtime because durable execution, HITL,
  verifier/goal loops, runtime budget policy, action firewall, persistent state,
  and multi-agent orchestration are absent or incomplete.

## What To Extract Into The Catalog

Recommended technique notes:

1. **Productized ReAct + MCP Assistant Platform**
   - Assistant config selects model, prompt, tools, sharing scope, and
     organizational access.

2. **Per-User MCP Tool Cache With Locking**
   - Redis cache keyed by user.
   - Redis lock for cache warm-up.
   - Partial failure short TTL.
   - No full TTL cache when every MCP source fails.

3. **MCP Tool Description Enrichment**
   - Custom configured descriptions override weak upstream MCP descriptions.

4. **Tool-Group Driven State Schema Selection**
   - Tool metadata drives which agent state/policy schema is used.

5. **Scoped Tool Policy For Domain Tool Groups**
   - Atlassian example maps Jira/Confluence/general scopes to tool filtering and
     prompt selection.
   - Current implementation appears not fully active because inference is
     commented out.

6. **Guarded Data-Source Context Injection**
   - Uploaded documents are XML-ish context blocks with explicit untrusted-data
     instructions and title citation guidance.

7. **Tool-Local Reflective Subgraph**
   - Simplify tool uses generate/critique/refine loop with a revision cap.

8. **Tool Progress Streaming**
   - Tools stream custom chunks that the backend maps into OpenAI-compatible
     streaming chunks.

## Highest-Impact Improvements

If MUCGPT wanted to move from B-/B to A-ish agentic runtime:

1. **Turn on or finish stateful scope routing**
   - Persist current scope.
   - Run scope inference only when needed.
   - Avoid re-inferring every turn.
   - Keep assistant prompt and append scope instructions instead of replacing.

2. **Add server-side durable conversation/runtime state**
   - LangGraph checkpointer or equivalent.
   - Thread/session IDs.
   - Replayable state.
   - Server-side context compaction.

3. **Add HITL approval for risky MCP tools**
   - Approve/edit/reject before external writes, sends, deletes, or ticket/page
     mutations.

4. **Add prompt-injection action firewall**
   - Track original user intent.
   - Label untrusted context.
   - Screen proposed tool calls before execution.

5. **Add budget policy**
   - Max turns.
   - Max tool calls.
   - Token/cost thresholds.
   - Hard-stop and graceful degradation.

6. **Add tool health telemetry**
   - Success/error rates.
   - Latency.
   - Schema mismatch.
   - Quarantine/fallback behavior.

7. **Make general answers verifier-aware**
   - Optional answer verifier for factual/research/tool-heavy tasks.
   - Source/citation check when InternetSearch or documents are used.

## Bottom Line

MUCGPT is a credible, product-oriented ReAct/MCP assistant platform. It is much
more mature than a demo chatbot because it has assistant configuration,
organizational access, MCP integration, streaming tool progress, Langfuse
tracing, and some policy middleware.

But its agentic runtime is not yet advanced in the sense of durable autonomous
agents. It lacks the hard runtime primitives that make long-running agents safe
and inspectable: persistent server-side state, explicit budgets, HITL gates,
action-level prompt-injection defenses, verifier loops, and multi-agent
coordination.
