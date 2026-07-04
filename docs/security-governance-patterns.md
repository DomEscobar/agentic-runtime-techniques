# Security and Governance Runtime Patterns

Status: 2026-07-04

These patterns help agents decide how to run safely, not just how to reason.
They cover hostile context, tool access, budgets, tool quality, provenance, and
session replay.

Sources:

- OWASP LLM01 Prompt Injection:
  https://genai.owasp.org/llmrisk/llm01-prompt-injection/
- OWASP LLM Prompt Injection Prevention Cheat Sheet:
  https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html
- OpenAI agent builder safety guide:
  https://developers.openai.com/api/docs/guides/agent-builder-safety
- OpenAI Guardrails prompt-injection detection:
  https://openai.github.io/openai-guardrails-python/ref/checks/prompt_injection_detection/
- NVIDIA NeMo Guardrails overview:
  https://docs.nvidia.com/nemo/guardrails/about-nemo-guardrails-library/overview
- Tau append-only session primitives:
  https://github.com/huggingface/tau/tree/main/src/tau_agent/session

## 1. Prompt-Injection Action Firewall

```text
user intent
    |
    v
untrusted content enters context
    |
    v
model proposes tool/action
    |
    v
action screening against original intent
    | allow        | block/escalate
    v             v
execute       human / safe refusal
```

Prompt injection is a runtime problem because the attack usually arrives through
retrieved/web/email/file content and becomes dangerous when the agent takes
downstream actions. The defense is not only "detect bad text"; it is to screen
each proposed action against the original user intent and the trust level of
the context that influenced it.

Runtime primitives:

- original user intent record
- trusted vs untrusted context labels
- proposed action record
- action screening guardrail
- block / allow / human-review route
- audit event

Use when:

- The agent reads untrusted content.
- The agent can call tools, send messages, edit files, buy things, browse while
  logged in, or access private data.

Failure modes:

- indirect prompt injection through retrieved content
- tool call drift away from user intent
- data exfiltration through downstream tools
- self-policing guardrail bypass

Decision note:

- If untrusted content can influence tools, add this before adding more
  autonomy.

## 2. Context Trust Zones

```text
context source
    |
    v
classify trust zone
    | system/user-owned | retrieved/untrusted | tool output | memory
    v                   v                     v             v
packing policy      quarantine labels     output policy   provenance
```

The runtime tracks where context came from and how much authority it has.
Untrusted content can be useful evidence, but should not be allowed to override
system instructions, user intent, or tool policy.

Runtime primitives:

- source labels
- trust zone metadata
- context packing rules
- quote/evidence boundaries
- policy for tool-influencing context
- provenance links

Use when:

- The agent mixes system instructions, user instructions, web pages, files,
  memory, tool outputs, and generated summaries.

Failure modes:

- untrusted text becomes instruction
- summary laundering hides malicious source text
- memory stores injected instruction as durable fact

Decision note:

- Long-context agents need trust-zone metadata more than they need bigger
  prompts.

## 3. Capability / Least-Privilege Runtime

```text
task intent
    |
    v
derive required capabilities
    |
    v
grant scoped tools/data
    |
    v
run agent
    |
    v
revoke / expire / audit
```

The agent should not receive every tool or secret by default. Capabilities are
granted for a task, scoped to the user intent, and revoked or expired when the
task ends.

Runtime primitives:

- capability manifest
- scoped tool grants
- data-access scopes
- approval policy
- expiry/revocation
- audit log

Use when:

- The agent can access private data, credentials, messaging surfaces, payment,
  code deployment, or destructive file operations.

Failure modes:

- excessive agency
- credential leakage
- accidental public action
- compromised tool chain

Decision note:

- Guardrails are not a substitute for least privilege. A model cannot leak or
  misuse tools it never received.

## 4. Runtime Budget Policy Engine

```text
run starts
    |
    v
budget policy
tokens / time / tool calls / cost / depth
    |
    v
warn threshold?
    | no          | yes
    v             v
continue       compress / ask / downshift
    |
    v
hard limit?
    | no          | yes
    v             v
continue       strip tools / stop / checkpoint
```

Budget policy is a runtime governor, not just a billing concern. It prevents
unbounded loops, runaway tool calls, excessive context, and accidental expensive
work.

Runtime primitives:

- token budget
- time budget
- tool-call budget
- cost budget
- depth/handoff budget
- warning threshold
- hard-stop action

Use when:

- The agent can loop, delegate, search, call expensive tools, run code, or keep
  background tasks alive.

Failure modes:

- unbounded consumption
- hidden cost spikes
- infinite delegation
- context inflation

Decision note:

- Every autonomous loop needs a budget policy before it needs more clever
  planning.

## 5. Tool Reliability Scoring and Fallback

```text
tool registry
    |
    v
execute tool
    |
    v
record outcome
success / error / latency / schema mismatch
    |
    v
update reliability score
    |
    v
route next call
primary / fallback / quarantine / human
```

Agents often treat tools as equally reliable. A runtime can track whether tools
fail, return malformed data, time out, violate schemas, or produce low-quality
results, then route future calls accordingly.

Runtime primitives:

- tool registry
- schema diagnostics
- success/error counters
- latency and timeout records
- fallback tools
- quarantine state
- human escalation route

Use when:

- Multiple tools can satisfy the same need.
- Tools are remote, flaky, provider-owned, user-installed, or schema-generated.

Failure modes:

- repeated broken tool calls
- silent schema drift
- low-quality tool output treated as fact
- bad tool chosen because it was available

Decision note:

- Tool selection should be based on capability and reliability, not just name
  matching.

## 6. Artifact Provenance Graph

```text
source / tool / model step
    |
    v
claim or artifact
    |
    v
provenance edge
source -> claim -> artifact -> final answer
    |
    v
audit / replay / citation
```

The runtime tracks which source, tool call, model step, or human action created
each claim and artifact. This is stronger than a flat citation list because it
preserves relationships.

Runtime primitives:

- artifact IDs
- claim IDs
- source IDs
- provenance edges
- transformation records
- final answer mapping
- redaction policy

Use when:

- Outputs need audit, citations, reproducibility, compliance, or debugging.

Failure modes:

- citation mismatch
- source laundering through summaries
- untraceable generated artifacts
- human cannot inspect why the answer was produced

Decision note:

- If an answer matters, store claim-to-evidence links, not just final text.

## 7. Append-Only Session Event Log / Branchable Replay

```text
event happens
    |
    v
append typed session entry
message / model_change / compaction / leaf / custom
    |
    v
replay entries
    |
    v
derive active state
    |
    v
optionally replay root-to-leaf branch
```

Tau's session primitives show a compact version of this pattern: sessions are
append-only JSONL entries; state is reconstructed by replay; compaction replaces
older message entries during replay; leaf entries support active branch
selection.

Runtime primitives:

- append-only entry storage
- typed event models
- parent IDs
- active leaf pointer
- root-to-leaf path reconstruction
- compaction entries
- custom extension entries

Use when:

- The agent needs durable sessions, branchable conversations, compaction,
  replay, or audit.

Failure modes:

- overwritten session history
- broken branch tree
- compaction without provenance
- inability to reconstruct active context

Decision note:

- Store raw durable events first, derive active context second.

## How These Fit The Decision Layer

Add these when the base loop becomes operational:

| Problem | Pattern |
| --- | --- |
| Agent reads hostile web/email/file content | Prompt-Injection Action Firewall |
| Context comes from mixed-trust sources | Context Trust Zones |
| Agent has powerful tools or secrets | Capability / Least-Privilege Runtime |
| Agent can loop, delegate, or spend money | Runtime Budget Policy Engine |
| Tools are flaky or overlapping | Tool Reliability Scoring and Fallback |
| Claims/artifacts must be audited | Artifact Provenance Graph |
| Sessions must resume, branch, or compact | Append-Only Session Event Log |
