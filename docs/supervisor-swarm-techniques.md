# Supervisor and Swarm Techniques

Status: 2026-07-04

Sources:

- OpenAI Swarm: https://github.com/openai/swarm
- LangGraph Swarm: https://github.com/langchain-ai/langgraph-swarm-py
- LangGraph Supervisor reference: https://reference.langchain.com/python/langgraph-supervisor
- LangChain multi-agent benchmark: https://www.langchain.com/blog/benchmarking-multi-agent-architectures
- CrewAI hierarchical process: https://docs.crewai.com/v1.15.0/en/concepts/processes

## Supervisor

Supervisor is a centralized multi-agent runtime architecture.

Shape:

```text
user -> supervisor -> worker agent -> supervisor -> user
```

Core runtime primitives:

- Central routing agent
- Worker/subagent registry
- Handoff/delegation tools
- Result return channel
- Optional direct forwarding tool
- Delegation ledger
- Handoff-message trimming
- Typed worker result contracts
- Recursion / continuation limits

Strengths:

- Generic: workers do not need to know about each other.
- Easier governance: one agent owns user-facing response and policy.
- Good when workers are third-party agents or domain-specific black boxes.
- Easier to add auditing, cost attribution, and permission boundaries.

Weaknesses:

- Translation loss: supervisor may paraphrase worker output incorrectly.
- Extra token cost: supervisor reads and rewrites more context.
- Bottleneck: central coordinator can become the weak link.
- Delegation loops need explicit breakers.

Catalog placement:

- Multi-agent orchestration
- Router/delegation
- Hierarchical runtime
- Result synthesis

Tier note:

- B/A boundary. It has public implementation evidence and a 2025 LangChain
  benchmark, but benchmark results depend heavily on message forwarding and
  context-management details.

## Swarm

Swarm is a decentralized handoff architecture.

Shape:

```text
user -> active agent -> handoff -> next active agent -> user
```

Core runtime primitives:

- Active agent state
- Agent-to-agent handoff tools
- Context variables passed between agents
- Last-active-agent memory
- Optional checkpointer/store
- Max turns
- Streaming/debug hooks

Strengths:

- Less translation overhead: active specialist can respond directly.
- Natural for many independent capabilities.
- Lightweight: OpenAI Swarm centers on `Agent`s and handoffs.
- LangGraph Swarm remembers the last active agent for later interactions.

Weaknesses:

- Weaker central coordination.
- Each agent may need awareness of other agents.
- Harder to enforce one global plan unless a supervisor is added.
- Handoff loops need turn limits and observability.

Catalog placement:

- Multi-agent handoff
- Active-agent runtime
- Decentralized routing
- Context handoff

Tier note:

- B/A boundary. LangChain's benchmark found swarm and supervisor both scale
  better than a single agent with many distractor domains; swarm avoided some
  supervisor translation cost in that setup. Still, generic swarm is not
  automatically better than custom domain workflows.

## Supervisor vs Swarm

| Dimension | Supervisor | Swarm |
| --- | --- | --- |
| Control | Central coordinator | Current active agent |
| User response | Usually supervisor only | Active agent can respond |
| Worker knowledge | Workers need not know peers | Agents often know handoff options |
| Governance | Easier central policy | More distributed policy |
| Main failure | Translation/paraphrase loss | Handoff drift / weak global ownership |
| Cost profile | Often higher due to coordinator | Often lower in handoff-heavy cases |
| Best fit | Third-party agents, strict governance, manager-worker teams | Independent specialists, direct handoffs, lower coordination overhead |

## Hybrid Pattern

Many serious runtimes should not pick only one. A useful hybrid:

```text
root supervisor
    |
    +--> domain swarm A
    |
    +--> domain swarm B
    |
    +--> specialist worker C
```

The supervisor owns:

- User contract
- Policy
- Task boundaries
- Completion criteria
- Final synthesis

The swarm owns:

- Local handoffs
- Specialist routing
- Fast direct responses inside one domain

This hybrid is especially relevant for agentic runtimes with skills, MCP tools,
custom agents, and third-party workers.
