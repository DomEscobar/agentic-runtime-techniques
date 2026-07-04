# Benchmark and Release-Date Tier List

Status: 2026-07-04

This is not a single leaderboard. The techniques below are evaluated on
different tasks: QA, coding, Minecraft, retrieval, software engineering, and
multi-agent orchestration. The tier is an evidence label, not a claim that one
technique always beats another.

## Tier Criteria

| Tier | Meaning |
| --- | --- |
| S | Strong primary-source benchmark evidence, clear reusable runtime technique, and broad downstream influence. |
| A | Good benchmark evidence or strong production/runtime relevance, but narrower scope or less direct comparability. |
| B | Important technique with weaker public benchmarks, mostly framework evidence, or still-emerging runtime semantics. |
| C | Useful pattern, but evidence is mostly anecdotal, product-specific, or lacks clean public evaluation. |

## S Tier

| Published | Technique | Runtime Area | Benchmark / Evidence Signal | Sources |
| --- | --- | --- | --- | --- |
| 2022-05 | Least-to-Most Prompting | Query decomposition | 99%+ on SCAN splits with few exemplars; strong decomposition evidence for hard compositional tasks. | [arXiv](https://arxiv.org/abs/2205.10625) |
| 2022-10 | ReAct | Core action loop | HotpotQA, FEVER, ALFWorld, WebShop; large gains on interactive benchmarks and reduced hallucination via tool observations. | [arXiv](https://arxiv.org/abs/2210.03629) |
| 2023-03 | Reflexion | Reflection / memory loop | HumanEval 91% pass@1 reported; gains across coding, decision-making, and reasoning tasks through verbal feedback memory. | [arXiv](https://arxiv.org/abs/2303.11366) |
| 2023-05 | Tree of Thoughts | Search / planning loop | Game of 24: 74% success versus 4% for GPT-4 chain-of-thought in the paper setup; also evaluated on creative writing and crosswords. | [arXiv](https://arxiv.org/abs/2305.10601), [GitHub](https://github.com/princeton-nlp/tree-of-thought-llm) |
| 2023-05 | Voyager | Lifelong embodied agent loop | Minecraft: 3.3x more unique items, 2.3x longer travel, and up to 15.3x faster milestone unlocks versus prior SOTA. | [arXiv](https://arxiv.org/abs/2305.16291), [Project](https://voyager.minedojo.org/) |
| 2024-02 | CodeAct | Executable-code action loop | API-Bank and a curated benchmark across 17 LLMs; up to 20% higher success rate over common action formats. | [arXiv](https://arxiv.org/abs/2402.01030), [GitHub](https://github.com/xingyaoww/code-act) |
| 2024-05 | SWE-agent / Agent-Computer Interface | Coding harness / ACI | SWE-bench 12.5% and HumanEvalFix 87.7% pass@1 at publication; demonstrated UI/ACI design impact. | [arXiv](https://arxiv.org/abs/2405.15793), [GitHub](https://github.com/swe-agent/swe-agent) |

## A Tier

| Published | Technique | Runtime Area | Benchmark / Evidence Signal | Sources |
| --- | --- | --- | --- | --- |
| 2022-10 | Self-Ask with Search | Query analysis / decomposition | Benchmarked on compositional QA; adds explicit follow-up questions and can route subquestions to search. | [arXiv](https://arxiv.org/abs/2210.03350), [GitHub](https://github.com/ofirpress/self-ask) |
| 2022-12 | HyDE | Context acquisition / retrieval | Improves zero-shot dense retrieval across web search, QA, fact verification, and multilingual retrieval tasks. | [arXiv](https://arxiv.org/abs/2212.10496), [GitHub](https://github.com/texttron/hyde) |
| 2023-02 | Toolformer | Tool-use learning | Shows self-supervised API-use learning across calculator, QA, search, translation, and calendar tools. | [arXiv](https://arxiv.org/abs/2302.04761) |
| 2023-05 | ReWOO | Planning without observation | Six public NLP benchmarks plus curated data; 5x token efficiency and 4% HotpotQA accuracy improvement reported. | [arXiv](https://arxiv.org/abs/2305.18323) |
| 2023-10 | Self-RAG | Adaptive retrieval / critique | Outperforms ChatGPT and retrieval-augmented Llama2-chat on open-domain QA, reasoning, fact verification, and long-form factuality/citation tasks. | [arXiv](https://arxiv.org/abs/2310.11511), [Project](https://selfrag.github.io/) |
| 2024-01 | Corrective RAG | Retrieval correction loop | Four datasets covering short- and long-form generation; improves RAG-based approaches through retrieval evaluation/correction. | [arXiv](https://arxiv.org/abs/2401.15884) |
| 2024-06 | CRAG Benchmark | RAG evaluation benchmark | Shows advanced LLMs at <=34% and straightforward RAG around 44% accuracy on a dynamic factual QA benchmark. | [arXiv](https://arxiv.org/abs/2406.04744), [GitHub](https://github.com/facebookresearch/CRAG/) |

## B Tier

| Published | Technique | Runtime Area | Benchmark / Evidence Signal | Sources |
| --- | --- | --- | --- | --- |
| 2023-08 | AutoGen multi-agent conversations | Multi-agent orchestration | Strong framework and example evidence across domains; benchmark signal is less clean because it is a general infrastructure layer. | [arXiv](https://arxiv.org/abs/2308.08155), [Docs](https://microsoft.github.io/autogen/0.2/docs/Use-Cases/agent_chat/) |
| 2023-08 | MetaGPT SOP agents | Multi-agent workflow / SOPs | Evaluated on collaborative software-engineering tasks; important role/SOP pattern but harder to isolate from framework choices. | [arXiv](https://arxiv.org/abs/2308.00352) |
| 2024+ | Ralph Loop | Bounded retry / fresh-context loop | High practical relevance for autonomous coding loops; public evidence is mostly implementation and practitioner writeups rather than formal benchmarks. | [Article](https://ghuntley.com/ralph/), [orchestrator list](https://github.com/andyrewlee/awesome-agent-orchestrators) |
| 2024+ | Implement / Review / Merge | Coding harness loop | Strong practical pattern across coding agents; formal evidence usually comes through SWE-bench-like systems rather than the loop alone. | [SWE-bench](https://www.swebench.com/), [SWE-agent](https://arxiv.org/abs/2405.15793) |
| 2025-03 | OpenAI Agents SDK primitives | Handoffs / guardrails / tracing / sessions | Production-oriented orchestration primitives; not a benchmark paper, but useful as runtime taxonomy evidence. | [OpenAI](https://openai.com/index/new-tools-for-building-agents/), [Docs](https://openai.github.io/openai-agents-python/) |
| 2024+ | LangGraph durable execution | Durable orchestration runtime | Durable execution, persistence, streaming, and human-in-the-loop are explicit runtime features; public benchmark evidence is indirect. | [Docs](https://docs.langchain.com/oss/python/langgraph/overview), [GitHub](https://github.com/langchain-ai/langgraph) |

## C Tier / Emerging

| Published | Technique | Runtime Area | Benchmark / Evidence Signal | Sources |
| --- | --- | --- | --- | --- |
| 2025+ | Deep Research-style loops | Research loop | Important product/runtime pattern; public evidence is product-level and benchmark details are not always reproducible. | [OpenAI](https://openai.com/index/introducing-deep-research/) |
| 2026-07 | Onyx programs / heartbeat checkpoint loop | Programmable agent runtime | Strong conceptual runtime primitive: `run`, `spawn`, typed state, sleep, checkpoint, budget/error semantics. Public benchmark evidence not yet established. | [X article](https://x.com/i/article/2073112080140689408) |
| 2026 | Dynamic workflow generation | Per-task harness synthesis | Promising direction: agent writes the harness/workflow for the task. Evidence is mostly product/demo-level so far. | [Anthropic news](https://www.anthropic.com/news/claude-opus-4-8) |
| varies | Planner / Executor harness | Multi-agent task routing | Common and useful; difficult to rank because implementations differ widely and benchmarks often measure the full agent stack. | [orchestrator list](https://github.com/andyrewlee/awesome-agent-orchestrators) |
| varies | Heartbeat / checkpoint loop | Long-running agent operations | Essential for assistants and background workers; usually evaluated operationally rather than with academic benchmarks. | [LangGraph docs](https://docs.langchain.com/oss/python/langgraph/overview), [Onyx article](https://x.com/i/article/2073112080140689408) |

## Cross-Cutting Techniques To Add Next

These belong in the repo but need more source work before tiering:

- Query routing and intent classification
- Context packing and compression
- Long-context state summarization
- Tool selection and tool reliability scoring
- Human approval gates
- Prompt-injection defense loops
- Runtime budget policies
- Task queues and resumable workflows
- Multi-agent communication protocols
- Agent observability, tracing, and run receipts

## Read This Carefully

Benchmarks are not comparable across rows. A 74% Game of 24 result, a 12.5%
SWE-bench result, and a 3.3x Minecraft exploration result measure different
worlds. The tier answers a narrower question:

> How strong is the public evidence that this is a reusable agentic runtime
> technique rather than just a prompt trick or product feature?
