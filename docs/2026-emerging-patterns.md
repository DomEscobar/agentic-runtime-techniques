# 2026 Emerging Runtime Patterns

Status: 2026-07-04

Source: DeerResearch sweep over 2026 arXiv preprints, archived in
`research/2026-arxiv-agentic-runtime-techniques/` (`report.md`, `evidence.jsonl`,
`arxiv_metadata.jsonl`, `claims.jsonl`, `sources.jsonl`), summarized in
`analysis/2026-arxiv-agentic-runtime-techniques.md`.

This is a companion deep-dive, not a second catalog. Every technique named
below has a canonical entry in `data/techniques.yml` and a tier row in
`data/tier-list.yml`; this document adds the narrative context, caveats, and
cross-references that don't fit in the YAML schema.

## Why This Pass Exists

2026 arXiv work moves past "agent = ReAct + tools" toward runtimes that
bound execution, admit/commit actions transactionally, optimize workflow
portfolios, enforce policy in the serving path, trace provenance, and
schedule whole workflows instead of isolated model calls. All ten papers
below are fresh 2026 preprints; treat their numbers as early evidence until
independently replicated, which is why every entry is capped at C tier in
`data/tier-list.yml`.

## New Catalog Entries

| Technique | Catalog Placement | What's New | Source |
| --- | --- | --- | --- |
| Infinite Agentic Loop Static Analysis | Bounded retry loop | Build/CI-time static analyzer: framework-independent agent IR plus a loop dependence graph finds feedback paths with no effective bound, before a run ever starts. 68 confirmed failures across 47 projects at 91.9% precision. | [arXiv:2607.01641](https://arxiv.org/abs/2607.01641) |
| Self-Healing Orchestrator | Tool runtime | Failure signals are classified by type, a targeted recovery action runs under a budget, and the recovered trajectory is verified before continuing, instead of blind retry or full replanning. 98.8% task success on controlled fault injection. | [arXiv:2606.01416](https://arxiv.org/abs/2606.01416) |
| Workflow Portfolio Router | Harness runtime | A bank of complementary workflows is precomputed offline; a router predicts the best-fit workflow per query instead of reasoning about structure from scratch. | [arXiv:2606.11290](https://arxiv.org/abs/2606.11290) |
| Meta-Tool Compiler | Tool runtime | Repeated tool-call sequences are mined from traces and compiled into deterministic composite tools, cutting LLM calls for recurring patterns. | [arXiv:2601.22037](https://arxiv.org/abs/2601.22037) |
| Cost-Aware Agentic Query Optimizer | Cost runtime | Agentic query execution is treated like database query optimization: operator placement, selectivity scope, and projection width become explicit knobs tuned against a quality-cost objective. | [arXiv:2606.03152](https://arxiv.org/abs/2606.03152) |
| Trajectory-Level Reward / Verifier | Test-time compute | Reward/verifier models score a whole tool-integrated trajectory rather than a final answer or a single step; benchmark shows all evaluator families degrade on long-horizon traces. | [arXiv:2604.08178](https://arxiv.org/abs/2604.08178) |
| Memory Write-Manage-Read Loop | Memory runtime | Memory becomes an explicit three-phase lifecycle: write via semantic structured compression, manage via online semantic synthesis, read via intent-aware retrieval planning, instead of flat top-k similarity. | [arXiv:2601.02553](https://arxiv.org/abs/2601.02553) |
| Agent-Aware Serving Policy Layer | Cost runtime | A middle layer between orchestration and the serving engine exposes observe/score/predict/act primitives so caching, batching, fairness, memoization, and speculative execution decisions see agent identity and workflow structure. Merges three independent papers making the same argument. | [arXiv:2605.27744](https://arxiv.org/abs/2605.27744), [arXiv:2603.16104](https://arxiv.org/abs/2603.16104), [arXiv:2605.16637](https://arxiv.org/abs/2605.16637) |

## Backfilled Sources on Existing Entries

Two entries in `data/techniques.yml` previously had `sources: []` with a note
saying no canonical primary source had been found yet. This sweep closed
both gaps:

- **Saga / Compensating Transaction Loop** — Mnemosyne's Agentic Transaction
  Processing (proposal -> admission -> commit -> repair, with an append-only
  transition log and dependency-safe compensation) is a more formal,
  academically-grounded treatment of the same pattern.
  ([arXiv:2607.00269](https://arxiv.org/abs/2607.00269))
- **Artifact Provenance Graph** — "From Agent Traces to Trust" defines
  execution provenance as a typed graph and evidence tracing as its
  projection onto support relations, connecting retrieval grounding,
  tool-use safety, memory lineage, debugging, audit, and recovery.
  ([arXiv:2606.04990](https://arxiv.org/abs/2606.04990))

**Episodic Vector-Store Memory Retrieval Loop** picked up a supporting
survey source ([arXiv:2603.07670](https://arxiv.org/abs/2603.07670)) but is
intentionally not claimed as fully backfilled: the survey covers flat
similarity retrieval as one point in a broader design space rather than
being a dedicated primary source for it. The more complete treatment lives
in the new sibling entry, Memory Write-Manage-Read Loop, above.

## Secondary Sources Folded Into Existing Entries

A few 2026 papers are surveys or domain-specific benchmarks. Per this
repo's sourcing rules, they support existing entries as additional evidence
rather than becoming standalone rows:

- **Autoresearch** gained a second, independent source: an aerospace
  control paper with the identical propose/run/evaluate/keep-revert shape,
  contributing the idea of a credibility layer that certifies improvements
  against measured seed noise before they're kept.
  ([arXiv:2606.20394](https://arxiv.org/abs/2606.20394))
- **Skill Library / Lifelong Skill-Acquisition Loop** gained the 2026 Agent
  Skills survey, which frames skills as reusable procedural runtime
  artifacts rather than prompt snippets.
  ([arXiv:2605.07358](https://arxiv.org/abs/2605.07358))
- **Guardrail Pipeline** gained "Governance by Construction," whose CUGA
  policy-as-code system intercepts at five structural checkpoints (upstream
  of planning, tool calls, and outputs, among others) rather than only as a
  final moderation pass.
  ([arXiv:2605.20874](https://arxiv.org/abs/2605.20874))
- **Plan-and-Execute** gained GeoAgentBench as a benchmark source: its
  Plan-and-React architecture separates a stable global plan from
  step-wise reactive execution and per-step repair, a domain-specific
  instance of the same pattern.
  ([arXiv:2604.13888](https://arxiv.org/abs/2604.13888))

## Explicitly Left Out

- **SAGE** (Challenger/Planner/Solver/Critic self-evolution,
  [arXiv:2603.15255](https://arxiv.org/abs/2603.15255)) does not cleanly map
  onto an existing entry and was not force-fit into one. Revisit if a future
  pass finds it converges with an existing or new multi-agent entry.
- General surveys covering agentic reasoning, tool-use orchestration, and
  trustworthy agentic AI support taxonomy and terminology but are not
  themselves reusable runtime techniques, per `CONTRIBUTING.md`.

## Practical Decision Notes

From the underlying DeepResearch report:

- If the agent can loop, recurse, retry, delegate, or hand off, add
  explicit loop-bound analysis and runtime stop conditions.
- If the agent changes durable state, treat model actions as proposals and
  put admission, commit, rollback, and compensation in the runtime.
- If tools fail often, use typed failure classification plus bounded
  recovery; blind retry is weaker than self-healing with verification.
- If the system is multi-agent or high-throughput, add a policy/runtime
  layer between framework and model server so caching, fairness, safety,
  batching, and memoization share agent identity.
- If users repeat similar tasks, mine traces into workflows, skills, and
  meta-tools instead of re-reasoning every time.
- If the system must be trusted, store provenance graphs and evidence links
  as first-class artifacts, not logs for later.

## Caveats

- Many 2026 papers are fresh preprints; treat numerical claims as early
  evidence until independently replicated.
- Several benchmarks are domain-specific (aerospace, GIS); only the
  reusable runtime primitive was extracted, not the whole domain system.
- This sweep is arXiv-first. It does not cover GitHub-only framework
  releases or product docs; those are tracked separately as evidence for
  existing entries when they appear (see `data/tier-list.yml`).
