# OpenClaw and Hermes Technique Notes

Status: 2026-07-04

Local inspection found OpenClaw as an installed runtime under
`/usr/lib/node_modules/openclaw` and local workspace/config/state under
`/root/.openclaw`. I did not find a separate local "Hermes agents" codebase.
The only local `hermes` filesystem hits were React Native Hermes parser/compiler
packages in `node_modules`.

However, one local agent-loop document explicitly references **Hermes adaptation**
for skill extraction and reinjection:

- `/root/.openclaw/media/inbound/agent-loop-flow---2167a633-95af-419c-a263-d952398e287a.md`

This document maps the agent loop into techniques that belong in this repo.

## OpenClaw Runtime Techniques Found

| Technique | Runtime Area | Local Evidence | Catalog Placement |
| --- | --- | --- | --- |
| Bootstrap context assembly | Context engineering | Workspace bootstrap files: `AGENTS.md`, `SOUL.md`, `USER.md`, `TOOLS.md`, `HEARTBEAT.md`, `MEMORY.md`; runtime exports `resolveBootstrapContextForRun`, `resolveBootstrapFilesForRun`, `buildBootstrapContextForFiles`. | Context Acquisition / Context Shaping |
| Context engine | Context lifecycle | Plugin SDK exports `ContextEngine`, `assembleHarnessContextEngine`, `bootstrapHarnessContextEngine`, `finalizeHarnessContextEngineTurn`, `runHarnessContextEngineMaintenance`. | Context Engine / State Runtime |
| Preemptive compaction | Context compression | Plugin SDK exports `shouldPreemptivelyCompactBeforePrompt`, `compactContextEngineWithSafetyTimeout`, `compactWithSafetyTimeout`, `PREEMPTIVE_OVERFLOW_ERROR_TEXT`. | Context Shaping / Compression |
| Tool schema normalization | Tool reliability | Plugin SDK exports `normalizeAgentRuntimeTools`, `normalizeProviderToolSchemas`, `inspectRuntimeToolInputSchemas`, `RuntimeToolSchemaDiagnostic`. | Tool Selection / Tool Contract Enforcement |
| Before/after tool hooks | Tool governance | Plugin SDK exports `runBeforeToolCallHook`, `wrapToolWithBeforeToolCallHook`, `runAgentHarnessAfterToolCallHook`, `hasBeforeToolCallPolicy`. | Guardrails / Tool Policy |
| Deferred tool approval | Human gate | Plugin SDK exports `requestDeferredPluginToolApproval`, `DeferredPluginToolApproval`, plus approval delivery runtime files. | Human-in-the-loop / Approval Gate |
| Session write lock | Concurrency control | Plugin SDK exports `acquireSessionWriteLock`, `resolveSessionWriteLockOptions`, `SessionWriteLockAcquireTimeoutConfig`. | Concurrency / Race Prevention |
| Subagent spawning | Delegation | Config types expose `sessions_spawn`, `subagents.maxConcurrent`, `maxSpawnDepth`, `maxChildrenPerAgent`, `runTimeoutSeconds`. Runtime exports subagent session cleanup, spawn planning, and embedded run controls. | Delegation / Multi-agent Orchestration |
| Spawn workspace isolation | Coding/runtime safety | Plugin SDK exports `resolveAttemptSpawnWorkspaceDir`, `resolveAttemptFsWorkspaceOnly`, sandbox workspace access helpers. | Isolated Execution / Coding Harness |
| Heartbeat loop | Durable background runtime | Config types expose interval, active hours, target, light context, isolated session, skip-when-busy. Runtime defines `HEARTBEAT_PROMPT`, `HEARTBEAT_OK`, `heartbeat_respond`. | Durable Runtime Loop |
| Heartbeat structured response | Checkpoint/status emission | `HeartbeatToolResponse` has `outcome`, `notify`, `summary`, `priority`, `nextCheck`; outcomes include `no_change`, `progress`, `done`, `blocked`, `needs_attention`. | Checkpoint / Operational Status |
| Cron runtime | Scheduled runs | Runtime files include `cron`, `server-cron`, `cron-store-runtime`, `cron-snapshot.runtime`. | Scheduler / Durable Task Runner |
| Goal loop | Termination semantics | Runtime files include `commands-goal`; Codex goal tools are available in this environment as `create_goal`, `get_goal`, `update_goal`. | Goal / Verifier / Termination |
| Memory search/runtime | Retrieval and memory | Runtime files include `memory-host-search`, `memory-search`, `memory-runtime`, `memory-state`, active memory extension. | Memory / Context Retrieval |
| Skill Workshop | Durable workflow learning | Runtime exposes `skill_workshop` and prompt section builder. Workshop proposals are used for reusable skills. | Skill Extraction / Durable Instruction Management |
| Delivery planning | Output routing | Runtime files include `delivery-plan`, `delivery-queue`, `delivery-target`, `pending-final-delivery`, `best-effort-delivery`. | Delivery / Output Routing |
| Stuck session recovery | Failure recovery | Runtime files include `diagnostic-stuck-session-recovery`, session locks, session snapshots, transcript repair. | Failure Recovery / Blocked State |
| Terminal outcome classification | Run termination | Plugin SDK exports `classifyAgentHarnessTerminalOutcome` and result classification types. | Termination Semantics |
| Agent event hooks | Observability | Plugin SDK exports `emitAgentEvent`, `onAgentEvent`, `emitSessionTranscriptUpdate`, `AgentEventPayload`. | Tracing / Observability |

## Hermes-Adapted Techniques Found

The local loop-flow document describes Hermes influence as:

| Technique | Runtime Area | Evidence | Catalog Placement |
| --- | --- | --- | --- |
| Skill extraction from successful planning runs | Memory / learning | Successful planning tasks with confidence and zero eval loops are saved as reusable patterns, capped at 10 skills. | Reflection / Skill Memory |
| Skill reinjection | Context shaping | Relevant memory skills are injected into future prompts, limited to the top 3. | Context Shaping / Prompt Assembly |
| Pattern memory over raw history | Memory compression | Stores reusable procedural patterns rather than only conversation transcript. | Long-term Memory / Procedure Mining |

## Agent Loop Document Techniques

The local agent-loop document itself is useful as a technique source:

| Technique | Shape | Why It Matters |
| --- | --- | --- |
| Context Assembly | load memory -> compress history -> build system prompt -> assemble messages | Makes context explicit before inference. |
| Intake and Routing | heuristic first, LLM only for ambiguity | Saves calls and routes tasks by type. |
| Verifiability Gate | only evaluate tasks with a rubric | Avoids circular evaluator loops on subjective tasks. |
| Planning Gate | skip plan for low complexity | Prevents over-orchestration. |
| External Fact Marking | model marks `[FACT: ...]` claims | Creates anchors for later trigger/evaluator checks. |
| Trigger Check | uncertainty, contradiction, coverage gap, tool mismatch, low confidence | Runs evaluator only when there is a signal. |
| Separate Evaluator Prompt | evaluate draft as if it were not self-written | Mitigates self-preference/self-correction bias. |
| Optimizer Loop | targeted edits from evaluator feedback | Avoids full rewrites and bounded quality ratcheting. |
| Circuit Breaker | max LLM calls and max eval loops | Hard stop against runaway cost and loops. |
| HITL Status | return `hitl` after unresolved uncertainty | Turns failure into explicit human review state. |

## What To Add To The Main Taxonomy

These are missing or underrepresented in the current taxonomy:

- Context engine lifecycle
- Preemptive compaction before prompt overflow
- Tool schema normalization and provider compatibility
- Tool-call policy hooks
- Deferred approval gates
- Session write locks
- Spawn depth and child limits
- Heartbeat structured checkpoint response
- Delivery planning and output routing
- Stuck-session recovery
- Skill extraction and reinjection
- Verifiability gate
- Trigger-gated evaluator/optimizer loop
- External fact marking

## Caveat

The OpenClaw runtime source is bundled JavaScript, so many findings are based on
file names and exported TypeScript declarations rather than readable source
modules. That is still useful for taxonomy extraction, but weaker than reading
the original TypeScript implementation.
