# DeepResearch Report: 2026 arXiv agentic runtime techniques

## Executive summary

Broad arXiv sweep, focused on 2026 papers that expose reusable agentic runtime mechanics rather than only product claims. The main signal: 2026 work is moving beyond "agent = ReAct + tools" toward runtimes that bound execution, admit/commit actions transactionally, optimize workflow portfolios, enforce policy in the serving path, trace provenance, and schedule whole workflows rather than isolated model calls.

The most implementation-ready additions for this repo are:

- Infinite Agentic Loop detection and loop-bounding.
- Agentic Transaction Processing: proposal/admission/commit/repair.
- Self-healing orchestrators with failure classification, bounded recovery, verification, and traces.
- Policy-driven runtime layers between agent framework and serving engine.
- Workflow optimization through portfolios, meta-tools, and cost-aware query planning.
- Provenance graphs and evidence tracing as first-class runtime artifacts.
- Memory as write/manage/read plus compression and intent-aware retrieval planning.
- Trajectory-level reward/verifier loops for full tool-using runs.

## Evidence-backed findings

1. Infinite loops are now a named runtime failure class.

`When Agents Do Not Stop` defines Infinite Agentic Loops as repeated model calls, tools, workflow transitions, or agent handoffs that can continue without an effective bound. Its IAL-Scan approach builds a framework-independent Agent IR and an Agentic Loop Dependence Graph, reporting 68 confirmed failures across 47 projects at 91.9% precision. Runtime implication: agent systems need loop bounds on feedback paths, not just ad hoc max-turn counters. Source: https://arxiv.org/abs/2607.01641

2. Generated actions should not be trusted until admitted by deterministic runtime constraints.

`Mnemosyne` introduces Agentic Transaction Processing, where model/agent-generated workflow actions are proposals until admitted under executable constraints. The runtime keeps an append-only transition log, effective-state projection, dependency-safe compensation, and active commitment records. Runtime implication: risky workflows need a transaction boundary around agent proposals. Source: https://arxiv.org/abs/2607.00269

3. Self-healing is stronger than blind retry or full replanning.

`Self-Healing Agentic Orchestrators` treats reliability as bounded runtime control: observe failure signals, infer failure classes, choose targeted recovery under budgets, verify recovered trajectories, and record observability traces. The paper reports 98.8% task success on controlled fault injection, above retry-only and full replanning baselines. Runtime implication: recovery should be typed and verified, not just "try again." Source: https://arxiv.org/abs/2606.01416

4. Agent serving needs a policy tier, not only a framework and model server.

`A Policy-Driven Runtime Layer for Agentic LLM Serving` argues that policies such as prefix caching, batch shaping, fairness, tool-result memoization, speculative execution, and safety need both agent identity and engine events. It proposes observe/score/predict/act primitives in a middle runtime layer and validates the idea with CacheSage. Runtime implication: production agent stacks should expose agent-aware policy hooks between orchestration and serving. Source: https://arxiv.org/abs/2605.27744

5. Workflow optimization is becoming portfolio- and trace-driven.

`FlowBank` builds a compact bank of complementary workflows and routes each query to the best predicted workflow. `Optimizing Agentic Workflows using Meta-tools` mines traces for repeated action sequences and compiles them into deterministic composite tools, reducing LLM calls and improving success rate in its benchmarks. Runtime implication: repeated agent paths should be harvested into reusable workflows, skills, and meta-tools. Sources: https://arxiv.org/abs/2606.11290 and https://arxiv.org/abs/2601.22037

6. Cost-aware agent execution looks like query optimization.

`Cost-Aware Optimization for Agentic Query Execution` formalizes agentic query execution where planning interleaves with execution and LLM-backed operator placement affects both quality and cost. Runtime implication: agent frameworks should make operator placement, selectivity scope, and projection width explicit optimization knobs. Source: https://arxiv.org/abs/2606.03152

7. Workflow-level serving is a separate layer of work.

`Helium` and `HexAGenT` both point to the same systems lesson: agent applications experience workflow latency and cross-call redundancy, not isolated model-call latency. Runtime implication: schedulers need access to workflow state, dependencies, KV/cache behavior, and branching structure. Sources: https://arxiv.org/abs/2603.16104 and https://arxiv.org/abs/2605.16637

8. Provenance is becoming the backbone for audit, debugging, and recovery.

`From Agent Traces to Trust` defines execution provenance as a typed graph of an agent execution and evidence tracing as the projection onto support relations. It connects retrieval grounding, tool-use safety, memory lineage, debugging, audit, and recovery. Runtime implication: run receipts, evidence graphs, and memory lineage should be standard artifacts. Source: https://arxiv.org/abs/2606.04990

9. Memory is not "more context"; it is a runtime loop.

`SimpleMem` proposes semantic structured compression, online semantic synthesis, and intent-aware retrieval planning, reporting large token reductions. The 2026 memory survey frames agent memory as a write/manage/read loop coupled with action. Runtime implication: durable agents need explicit memory lifecycle policies, not just vector search. Sources: https://arxiv.org/abs/2601.02553 and https://arxiv.org/abs/2603.07670

10. Evaluation is shifting from final answers to trajectories.

`Plan-RewardBench` evaluates reward models over full tool-integrated trajectories and shows degradation on long-horizon traces. `GeoAgentBench` uses a dynamic execution sandbox and a Plan-and-React architecture that separates global orchestration from step-wise reactive execution. Runtime implication: agent quality checks need full-run validators, execution sandboxes, and last-attempt alignment metrics. Sources: https://arxiv.org/abs/2604.08178 and https://arxiv.org/abs/2604.13888

## Recommendation

Add these as new candidate patterns in the main catalog after a second pass:

- `Infinite Agentic Loop Scanner`
- `Agentic Transaction Processing`
- `Self-Healing Orchestrator`
- `Policy-Driven Agent Runtime Layer`
- `Workflow Portfolio Router`
- `Meta-Tool Compiler`
- `Cost-Aware Agent Query Optimizer`
- `Workflow-Aware Serving Scheduler`
- `Execution Provenance Graph`
- `Memory Write-Manage-Read Loop`
- `Trajectory-Level Reward / Verifier`

Suggested tiering:

- Strong implementation/runtime signal: IAL-Scan, Mnemosyne/ATP, Self-Healing Orchestrator, Policy Runtime Layer, Meta-Tools, FlowBank, Helium/HexAGenT.
- Useful design taxonomy: Agentic Reasoning survey, Tool-Use Orchestration survey, Memory survey, Trustworthy Agentic AI survey, Agent Skills survey.
- Domain-specific but reusable: GeoAgentBench Plan-and-React, AutoResearch credibility layer, SafetyALFRED safety-conscious planning.

## Caveats / blockers

- Many 2026 papers are fresh preprints. Treat numerical claims as early evidence until independently replicated.
- Some papers are surveys; they support taxonomy and terminology but not a standalone runtime implementation.
- Several benchmarks are domain-specific. Only extract the reusable runtime primitive, not the whole domain system.
- This report is intentionally arXiv-first. It does not cover GitHub-only framework releases, product docs, or unpublished runtime systems.

## Sources

- https://arxiv.org/abs/2607.01641 — When Agents Do Not Stop: Uncovering Infinite Agentic Loops in LLM Agents
- https://arxiv.org/abs/2607.00269 — Mnemosyne: Agentic Transaction Processing for Validating and Repairing AI-generated Workflows
- https://arxiv.org/abs/2606.01416 — Self-Healing Agentic Orchestrators for Reliable Tool-Augmented Large Language Model Systems
- https://arxiv.org/abs/2605.27744 — A Policy-Driven Runtime Layer for Agentic LLM Serving
- https://arxiv.org/abs/2606.03152 — Cost-Aware Optimization for Agentic Query Execution
- https://arxiv.org/abs/2606.11290 — FlowBank: Query-Adaptive Agentic Workflows Optimization through Precompute-and-Reuse
- https://arxiv.org/abs/2601.22037 — Optimizing Agentic Workflows using Meta-tools
- https://arxiv.org/abs/2606.04990 — From Agent Traces to Trust
- https://arxiv.org/abs/2601.02553 — SimpleMem: Efficient Lifelong Memory for LLM Agents
- https://arxiv.org/abs/2603.07670 — Memory for Autonomous LLM Agents
- https://arxiv.org/abs/2604.08178 — Aligning Agents via Planning: Plan-RewardBench
- https://arxiv.org/abs/2604.13888 — GeoAgentBench
- https://arxiv.org/abs/2603.16104 — Efficient LLM Serving for Agentic Workflows
- https://arxiv.org/abs/2605.16637 — HexAGenT
- https://arxiv.org/abs/2603.15255 — SAGE
- https://arxiv.org/abs/2605.20874 — Governance by Construction for Generalist Agents
- https://arxiv.org/abs/2605.07358 — Agent Skills survey
- https://arxiv.org/abs/2606.20394 — Agentic AutoResearch
