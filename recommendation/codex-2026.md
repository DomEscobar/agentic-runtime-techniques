# Codex 2026 Agentic Chat Runtime Recommendation

## Position

An agentic chat should not be built as "one model call answers one message." It should be built as a small runtime around the conversation: route the message, load the right operating method, gather targeted context, execute through a controlled loop, verify the result, update memory when useful, and then reply.

The core shape:

```text
User Message
  -> Intent Router
  -> Skill Resolver
  -> Context Builder
  -> Risk Gate
  -> Planner
  -> Agent Loop
  -> Verifier / Critic
  -> Durable Memory
  -> Final Response
```

For longer work, the same flow can become asynchronous:

```text
Chat Turn
  -> TaskFlow Job
  -> Worker Sessions / Subagents
  -> Periodic Status
  -> Verification
  -> Final Delivery
```

## Runtime Stages

### 1. Intent Router

The first step is to classify what kind of turn this is.

Common intents:

- direct chat
- research
- coding
- code review
- debugging
- planning
- external action
- memory update
- reminder or scheduled task
- browser workflow
- deployment

The router should choose the lightest execution mode that can satisfy the request. Not every message deserves a plan, tools, subagents, or a long loop.

### 2. Skill Resolver

Skills should be loaded early, directly after intent detection.

```text
Message -> Intent -> Skill Match -> Context -> Risk -> Act
```

A skill is not a worker. A skill is a stored operating method: a reusable playbook for how to perform a class of task.

Examples:

- DeepResearch skill: plan sources, fetch primary material, extract evidence, cluster claims, write a sourced report.
- Coding skill: inspect the repo, identify local patterns, edit minimally, run tests, inspect the diff.
- Browser skill: handle tabs, stale references, login state, screenshots, and timeout recovery.
- Deployment skill: inspect state first, build, deploy, healthcheck, and preserve existing configuration.
- Review skill: lead with findings, use file and line references, prioritize bugs and regressions.

Skills stabilize behavior. They prevent the agent from reinventing its workflow on every turn.

### 3. Context Builder

The context builder should gather only the context that matters for the current turn.

Useful context sources:

- recent conversation
- relevant memory
- repository files
- project documentation
- tool outputs
- web or official docs
- previous task artifacts

The failure mode to avoid is loading everything. The agent should retrieve targeted context, not flood itself with irrelevant history.

### 4. Risk Gate

Before acting, the runtime should classify the action risk.

Safe internal actions:

- read files
- inspect repositories
- run tests
- analyze logs
- draft text
- edit local project files when requested

Risky or external actions:

- sending messages
- posting publicly
- deploying
- deleting data
- spending money
- modifying system services
- changing schedulers or infrastructure

Risky actions should require either explicit user approval or a narrow pre-approved policy.

### 5. Planner

Planning should be conditional. Small tasks should be handled directly. Larger tasks need a concise plan that can actually drive execution.

The planner should answer:

- What is the goal?
- What evidence or state is needed?
- Which skill applies?
- Is a loop required?
- Are subagents useful?
- What are the stop conditions?

## The Agent Loop

The loop lives in the execution layer, after intent, skill, context, and risk setup.

```text
while not done and budget_ok:
  observe()
  decide()
  act()
  evaluate()
  update_working_state()
```

Expanded:

```text
Agent Loop
  1. Observe current state
  2. Decide next action
  3. Use a tool, delegate, edit, search, test, or draft
  4. Inspect the result
  5. Update working memory
  6. Check stop conditions
  7. Repeat if needed
```

The loop is not "think again until something sounds good." It is a controlled observe-act-evaluate cycle with budget, skill rules, and explicit stop criteria.

### Stop Conditions

A loop should stop when one of these is true:

- the user goal is satisfied
- enough evidence has been collected
- tests or validation pass
- the remaining action requires user approval
- the environment is blocked
- budget or timeout is reached
- the loop detects repeated failed actions
- the result is good enough for the requested depth

Without stop conditions, agentic systems drift into expensive wandering.

## Skills Inside The Loop

Skills shape the loop. Different skills imply different observe-act-evaluate cycles.

Coding loop:

```text
read repo
  -> identify relevant files
  -> edit
  -> run tests
  -> inspect diff
  -> if failing: diagnose, edit, test again
  -> if passing: summarize
```

Research loop:

```text
search
  -> fetch sources
  -> extract evidence
  -> cluster claims
  -> identify gaps
  -> search again if needed
  -> confidence check
  -> write report
```

Review loop:

```text
inspect diff
  -> trace behavior
  -> check tests
  -> identify regressions
  -> rank findings by severity
  -> report with file and line references
```

The same runtime can support many behaviors if skills define the local loop policy.

## Subagents

Subagents are delegated workers. They should be used inside the loop when work is large, parallelizable, or benefits from independent analysis.

Subagents are useful when:

- the task has clearly separable parts
- broad research is needed
- a second perspective reduces risk
- several modules must be understood in parallel
- long-running work should happen asynchronously
- a critic or verifier should independently challenge the result

Subagents are not useful for small tasks. They add coordination overhead and can make simple work slower.

Subagent execution pattern:

```text
decide()
  -> spawn subagent A / B / C
  -> wait for results
  -> merge findings
  -> resolve contradictions
  -> verifier checks final synthesis
```

Example roles:

- Research Agent
- Code Inspection Agent
- Test Strategy Agent
- Critic Agent
- Migration Agent
- UI Screenshot Audit Agent

The parent agent remains responsible for synthesis and final judgment. Subagents produce evidence and partial conclusions, not the final truth by default.

## Recommended Execution Modes

### Direct Mode

Use for simple conversation, explanations, and low-risk answers.

```text
Intent -> Context if needed -> Answer
```

### Single-Agent Tool Loop

Use for normal coding, debugging, repo edits, focused research, and local automation.

```text
Intent -> Skill -> Context -> Risk -> Loop -> Verify -> Reply
```

### Subagent Fan-Out

Use for broad research, multi-module audits, migrations, and tasks where independent perspectives are valuable.

```text
Intent -> Skill -> Plan -> Spawn Subagents -> Collect -> Critic -> Reply
```

### Durable Async TaskFlow

Use when work is long-running, scheduled, waiting on external state, or should survive outside the current chat turn.

```text
Intent -> TaskFlow Job -> Worker Sessions -> Status Updates -> Final Delivery
```

## Practical Recommendation

The best default architecture is:

```text
Message
  -> Routing / Setup
  -> Skill-guided Agent Loop
  -> Verification
  -> Reply
```

With optional expansion:

```text
Message
  -> Intent Router
  -> Skill Resolver
  -> Context Builder
  -> Risk Gate
  -> Planner
  -> Agent Loop
       -> Tool Calls
       -> Subagents when useful
       -> Working State Updates
       -> Stop Condition Checks
  -> Verifier / Critic
  -> Durable Memory Update
  -> Final Response
```

The key design principle: agentic depth should be proportional to task complexity.

A strong agent should first ask:

1. Can I answer directly?
2. Is there a skill for this?
3. What context is actually needed?
4. Is a loop necessary?
5. Would subagents improve the outcome enough to justify coordination cost?
6. What proves I am done?

Skills are almost always valuable because they make behavior repeatable. Subagents are valuable only when the work benefits from decomposition or independent verification. The loop is the execution engine that connects both.
