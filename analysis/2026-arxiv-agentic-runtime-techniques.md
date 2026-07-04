# 2026 arXiv Agentic Runtime Technique Sweep

This is the repo-facing summary of the DeepResearch artifact in
`research/2026-arxiv-agentic-runtime-techniques/`.

## Rating Signal

The 2026 arXiv signal is strong enough to add a new "emerging runtime
techniques" lane to this repo. The field is moving from simple tool loops toward
runtime systems that are bounded, transactional, policy-driven,
workflow-optimized, and provenance-aware.

## Best New Runtime Candidates

| Candidate | Why it matters | Source |
| --- | --- | --- |
| Infinite Agentic Loop Scanner | Detects unbounded feedback paths across model calls, tools, workflow transitions, and handoffs. | https://arxiv.org/abs/2607.01641 |
| Agentic Transaction Processing | Treats generated actions as untrusted proposals until admitted by deterministic constraints. | https://arxiv.org/abs/2607.00269 |
| Self-Healing Orchestrator | Maps failure signals to recovery actions under budgets, then verifies the recovered trajectory. | https://arxiv.org/abs/2606.01416 |
| Policy-Driven Runtime Layer | Adds observe/score/predict/act primitives between agent framework and serving engine. | https://arxiv.org/abs/2605.27744 |
| Workflow Portfolio Router | Precomputes complementary workflows and routes queries to the best reusable candidate. | https://arxiv.org/abs/2606.11290 |
| Meta-Tool Compiler | Mines repeated tool-call traces and compiles them into deterministic composite tools. | https://arxiv.org/abs/2601.22037 |
| Cost-Aware Agent Query Optimizer | Optimizes operator placement, granularity, and execution mode against quality-cost feedback. | https://arxiv.org/abs/2606.03152 |
| Workflow-Aware Serving Scheduler | Schedules whole agent workflows rather than isolated LLM calls. | https://arxiv.org/abs/2603.16104 |
| Execution Provenance Graph | Emits typed execution/evidence graphs for audit, debugging, memory lineage, and recovery. | https://arxiv.org/abs/2606.04990 |
| Memory Write-Manage-Read Loop | Treats memory as compression, synthesis, and intent-aware retrieval planning. | https://arxiv.org/abs/2601.02553 |

## Practical Decision Notes

- If the agent can loop, recurse, retry, delegate, or hand off, add explicit
  loop-bound analysis and runtime stop conditions.
- If the agent changes durable state, treat model actions as proposals and put
  admission, commit, rollback, and compensation in the runtime.
- If tools fail often, use typed failure classification plus bounded recovery;
  blind retry is weaker than self-healing with verification.
- If the system is multi-agent or high-throughput, add a policy/runtime layer
  between framework and model server so caching, fairness, safety, batching, and
  memoization share agent identity.
- If users repeat similar tasks, mine traces into workflows, skills, and
  meta-tools instead of re-reasoning every time.
- If the system must be trusted, store provenance graphs and evidence links as
  first-class artifacts, not logs for later.

## What To Add Next

The strongest next repo update would be a dedicated 2026 emerging-patterns doc
and YAML file covering:

- Infinite Agentic Loop Scanner
- Agentic Transaction Processing
- Self-Healing Orchestrator
- Policy-Driven Agent Runtime Layer
- Workflow Portfolio Router
- Meta-Tool Compiler
- Cost-Aware Agent Query Optimizer
- Workflow-Aware Serving Scheduler
- Execution Provenance Graph
- Memory Write-Manage-Read Loop
- Trajectory-Level Reward / Verifier

