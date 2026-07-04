# DeepResearch Report: agentic runtime techniques loops benchmarks release dates

## Executive summary

I researched primary papers, official docs, project pages, and GitHub
collections for agentic runtime techniques: query analysis, context acquisition,
planning, action loops, reflection, verification, multi-agent orchestration,
durable execution, and coding harnesses.

The result was added to the repo as:

- `docs/tier-list.md`
- `data/tier-list.yml`

The tier list ranks evidence strength and runtime relevance, not universal
performance. Benchmarks differ too much across QA, SWE-bench, Minecraft, RAG,
and multi-agent orchestration to form one absolute leaderboard.

## Evidence-backed findings

S-tier techniques have strong primary benchmark evidence and reusable runtime
shape:

- Least-to-Most Prompting: compositional query decomposition with strong SCAN
  results ([arXiv](https://arxiv.org/abs/2205.10625)).
- ReAct: core reason-act-observe loop benchmarked on HotpotQA, FEVER,
  ALFWorld, and WebShop ([arXiv](https://arxiv.org/abs/2210.03629)).
- Reflexion: verbal feedback memory, including 91% pass@1 on HumanEval in the
  paper report ([arXiv](https://arxiv.org/abs/2303.11366)).
- Tree of Thoughts: search/planning over thought branches, including 74% on
  Game of 24 in the paper setup ([arXiv](https://arxiv.org/abs/2305.10601)).
- Voyager: lifelong embodied agent loop in Minecraft with strong exploration
  and milestone metrics ([arXiv](https://arxiv.org/abs/2305.16291)).
- CodeAct: executable-code action space with up to 20% higher success rate on
  API-Bank/curated benchmarks ([arXiv](https://arxiv.org/abs/2402.01030)).
- SWE-agent / ACI: coding harness and agent-computer interface benchmarked on
  SWE-bench and HumanEvalFix ([arXiv](https://arxiv.org/abs/2405.15793)).

A-tier techniques have good evidence but are narrower or less directly
comparable:

- Self-Ask with Search ([arXiv](https://arxiv.org/abs/2210.03350)).
- HyDE ([arXiv](https://arxiv.org/abs/2212.10496)).
- Toolformer ([arXiv](https://arxiv.org/abs/2302.04761)).
- ReWOO ([arXiv](https://arxiv.org/abs/2305.18323)).
- Self-RAG ([arXiv](https://arxiv.org/abs/2310.11511)).
- Corrective RAG ([arXiv](https://arxiv.org/abs/2401.15884)).
- CRAG Benchmark ([arXiv](https://arxiv.org/abs/2406.04744)).

B/C-tier items are still important for a runtime-techniques repo, but evidence
is more indirect, framework-level, or product/demo-oriented:

- AutoGen ([arXiv](https://arxiv.org/abs/2308.08155)).
- MetaGPT ([arXiv](https://arxiv.org/abs/2308.00352)).
- LangGraph durable execution ([docs](https://docs.langchain.com/oss/python/langgraph/overview)).
- OpenAI Agents SDK primitives ([docs](https://openai.github.io/openai-agents-python/)).
- Ralph Loop and autonomous loop runners ([orchestrator list](https://github.com/andyrewlee/awesome-agent-orchestrators)).
- Onyx programmable runtime loops ([X article](https://x.com/i/article/2073112080140689408)).

## Recommendation

Use `docs/tier-list.md` as the public-facing version and `data/tier-list.yml`
as the structured source for later website/table generation.

The repo should keep two axes separate:

1. **Benchmark evidence**: paper-quality, reproducible evaluation where possible.
2. **Runtime importance**: primitives that matter operationally even when no
   clean public benchmark exists, such as durable execution, checkpoints,
   handoffs, guardrails, task queues, and human approval gates.

This separation is important because many of the most useful runtime techniques
are engineering primitives rather than standalone benchmark methods.

## Caveats / blockers

- Benchmarks are not comparable across domains.
- Framework papers often evaluate a whole system, not a single loop.
- Some current product/runtime techniques have official docs but no public
  benchmark.
- OpenAI blog URLs returned HTTP 403 to the local artifact fetcher; official
  docs and browser/search results were used as source support.
- The Onyx X article is difficult to fetch without JavaScript; the article was
  previously accessible through the X/vxTwitter context in this conversation.

## Sources

- https://arxiv.org/abs/2205.10625
- https://arxiv.org/abs/2210.03629
- https://arxiv.org/abs/2210.03350
- https://arxiv.org/abs/2212.10496
- https://arxiv.org/abs/2302.04761
- https://arxiv.org/abs/2303.11366
- https://arxiv.org/abs/2305.10601
- https://arxiv.org/abs/2305.16291
- https://arxiv.org/abs/2305.18323
- https://arxiv.org/abs/2308.08155
- https://arxiv.org/abs/2308.00352
- https://arxiv.org/abs/2310.11511
- https://arxiv.org/abs/2401.15884
- https://arxiv.org/abs/2402.01030
- https://arxiv.org/abs/2405.15793
- https://arxiv.org/abs/2406.04744
- https://docs.langchain.com/oss/python/langgraph/overview
- https://openai.github.io/openai-agents-python/
- https://www.swebench.com/
- https://github.com/nibzard/awesome-agentic-patterns
- https://github.com/andyrewlee/awesome-agent-orchestrators
