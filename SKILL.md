---
name: ai-task-router
description: |
  Cost-aware task routing and reasoning-governance skill for AI agents. Use at task intake before non-trivial work when execution environment, model/reasoning depth, tool use, file breadth, repository operations, research scope, or agent loops materially affect cost or quality. Routes work to the minimum sufficient environment, chooses the lowest sufficient reasoning depth, defines escalation and handoff rules, and stops redundant scans/reviews/agent loops unless the next pass has a concrete incremental-value target. Designed to be portable across ChatGPT, Work/workspace-style environments, Codex/code agents, and other Agent Skills-compatible systems.
---

# AI Task Router

> Route first. Reason second. Stop when marginal value hits zero.

## 0. Mission

This skill governs **how AI work is executed before the expensive work begins**.

Its objective is not to minimize tokens at all costs. Its objective is:

> **Meet the required quality bar with the lowest sufficient execution cost, while ensuring every expensive reasoning step creates measurable incremental value.**

Cost includes more than tokens:
- reasoning depth
- context loaded
- number of files scanned
- tool/API calls
- agent iterations
- repeated verification
- latency
- human interruption
- side-effect risk

Do not optimize cost by lowering required quality. Do not optimize quality by defaulting every task to the most expensive mode.

---

## 1. Operating principle

Before non-trivial execution, determine four things internally:

1. **Route** — where should the work happen?
2. **Reasoning budget** — how much reasoning is sufficient?
3. **Execution boundary** — what should be solved now versus handed off?
4. **Stop condition** — what evidence means the task is done?

If the current environment is already appropriate, proceed without bothering the user with routing commentary.

If a materially better environment is required and the agent cannot switch environments automatically, notify the user **before expensive execution begins**.

Use a concise handoff message such as:

`建议：CODE_AGENT + standard reasoning。原因：需要修改仓库、运行测试并验证结果；当前聊天只适合先闭合任务定义。`

Do not perform substantial work in a knowingly wrong environment and only explain the routing mistake afterward.

---

## 2. Canonical execution environments

Use platform-neutral route classes first. Product-specific names are adapters, not the core logic.

### 2.1 CONVERSATION
Best for:
- discussion and judgment
- ideation and critique
- short-form writing and rewriting
- clarifying requirements
- lightweight research synthesis
- single-output answers
- small calculations or transformations
- defining a closed specification before handoff

Typical ChatGPT adapter: **Chat**.

### 2.2 WORKSPACE
Best for:
- persistent multi-file work
- long-running artifact creation
- multi-document synthesis
- research requiring many sources and retained context
- iterative document/spreadsheet/presentation production
- tasks where project state must survive across steps

Typical ChatGPT adapter: **Work / persistent workspace**.

### 2.3 CODE_AGENT
Best for:
- repository inspection
- code edits
- terminal commands
- tests, linting, builds, migrations
- multi-file implementation
- deterministic file operations
- repeated edit → run → verify loops

Typical OpenAI adapter: **Codex**.

### 2.4 HYBRID
Use when the task has both high ambiguity and high execution cost.

Pattern:
1. Resolve intent, architecture, scope, and acceptance criteria in CONVERSATION.
2. Convert the result into a closed execution brief.
3. Handoff deterministic execution to WORKSPACE or CODE_AGENT.

HYBRID is often the cheapest high-quality route because it prevents expensive agents from discovering requirements through trial and error.

---

## 3. Route decision tree

Apply hard rules before soft judgment.

### Rule A — Repository or executable verification
If the task requires any of the following, prefer CODE_AGENT:
- editing repository files
- running shell commands
- tests/builds/lint/type-checks
- dependency changes
- code search across a repository
- migrations or generated code
- verifying implementation against runtime behavior

Exception: if the user is only asking for conceptual code advice or a tiny isolated snippet, CONVERSATION is sufficient.

### Rule B — Persistent multi-artifact state
Prefer WORKSPACE when two or more of the following are true:
- several files/documents must be created or maintained
- the work spans multiple stages
- source material is large
- later steps depend on earlier artifacts
- persistent project context materially reduces re-reading
- iterative editing across artifacts is expected

### Rule C — Ambiguity before execution
If the task is expensive to execute and the specification is materially ambiguous, do **not** send an expensive agent into discovery mode by default.

Use HYBRID:
- define the problem in CONVERSATION
- lock scope and acceptance criteria
- then hand off execution

### Rule D — Simple single-output work
Use CONVERSATION when the task can be completed correctly from the current context with a small number of reasoning/tool steps and does not require persistent state or repository execution.

### Rule E — Side effects
For actions with external side effects, choose the environment that provides the clearest verification and rollback path, not merely the shortest path.

Examples:
- code deployment → CODE_AGENT with tests/checks
- large document release → WORKSPACE with final QA
- sending a short message → CONVERSATION/app action if directly supported

---

## 4. Reasoning budget

Choose the **lowest reasoning tier that is sufficient**.

### R0 — Direct
Use for:
- extraction
- formatting
- deterministic transformations
- simple lookup
- obvious tool execution
- well-specified routine tasks

Do not invent complexity.

### R1 — Standard
Use for:
- moderate trade-offs
- multi-step planning
- normal design decisions
- synthesis across a limited number of inputs
- routine debugging where the failure surface is bounded

This is the default for non-trivial tasks.

### R2 — Deep
Reserve for:
- architecture with interacting constraints
- high ambiguity and high consequence
- conflicting evidence
- subtle causal diagnosis
- security/safety-sensitive reasoning
- complex strategic choices
- problems where a wrong assumption would invalidate major downstream work

### Escalation rule
Never start at R2 merely because it is available.

Escalate only when at least one condition holds:
- R0/R1 produced contradictory or incomplete conclusions
- critical uncertainty remains unresolved
- verification failed for a non-obvious reason
- the cost of a wrong answer is materially higher than the cost of deeper reasoning
- new evidence expands the problem rather than merely confirming it

### De-escalation rule
Once uncertainty is resolved and execution becomes deterministic, reduce reasoning depth for subsequent mechanical steps.

---

## 5. Incremental Value Gate

This is the central anti-waste rule.

Before any additional expensive pass, the agent must be able to name the **new value target** of that pass.

A pass is justified only if it is expected to produce at least one of:

- **E — New Evidence**: new facts, source material, measurements, logs, or observations
- **D — New Decision**: a previously unresolved choice is closed
- **A — New Artifact Delta**: the deliverable materially changes
- **V — New Verification**: a distinct claim, test, invariant, or acceptance criterion is verified
- **R — Risk Reduction**: a concrete failure mode is eliminated or bounded

If the next pass cannot reasonably produce E, D, A, V, or R, **stop**.

### Forbidden low-value loops
Do not repeat these without a new delta target:
- “scan everything again”
- “review once more”
- “double-check” with no distinct check
- rereading unchanged context
- rewriting the same artifact without a specified defect
- re-running the same failed action without changing an input, hypothesis, or environment
- letting an agent continue merely because budget remains

### Review passes must be orthogonal
Multiple reviews are allowed only when each review tests a different dimension.

Valid example:
1. correctness review
2. security review
3. performance review

Invalid example:
1. comprehensive review
2. comprehensive review again
3. one more comprehensive review

---

## 6. Context loading discipline

Context is a budget.

Load only what is required to answer the current question or execute the current step.

### Progressive disclosure
1. Start with task-local context.
2. Load indexes/summaries before full files when possible.
3. Open additional files only when a live uncertainty requires them.
4. Do not rescan unchanged files unless the previous scan was incomplete or a new question requires a different slice.

### Cache by conclusion
When a fact has already been established from stable material, reuse the conclusion rather than repeatedly paying to rediscover it.

Invalidate cached conclusions only when:
- source content changed
- the task asks a different question of the same source
- previous evidence was insufficient
- a contradiction appears

---

## 7. Define-before-delegate protocol

Expensive agents should receive closed tasks whenever possible.

Before handoff, produce an internal or explicit execution brief containing:

- Objective
- In-scope work
- Out-of-scope work
- Inputs/source of truth
- Locked constraints
- Deliverables
- Acceptance criteria
- Verification commands/checks
- Stop condition

If ambiguity can be resolved cheaply in CONVERSATION, resolve it before CODE_AGENT/WORKSPACE execution.

Do not make an expensive execution agent repeatedly ask strategic questions that could have been decided upstream.

---

## 8. Verification budget

Verification must match failure risk.

### Lightweight verification
Use when:
- output is reversible
- error impact is low
- task is deterministic

Examples: spelling check, file existence, basic formula check.

### Standard verification
Use when:
- multiple constraints interact
- output will be reused
- there are meaningful downstream dependencies

Examples: consistency checks, targeted tests, source cross-checks.

### Strong verification
Use when:
- external side effects exist
- financial/legal/safety/security consequences are meaningful
- deployment or data mutation occurs
- failure is hard to reverse

Strong verification does **not** mean repeating the same check. Use independent evidence or orthogonal tests.

---

## 9. Stop conditions

Stop when all required conditions are true:

1. Deliverable exists.
2. Acceptance criteria are satisfied.
3. Required verification has passed or remaining uncertainty is explicitly bounded.
4. No unresolved blocker prevents use.
5. The next plausible pass has no clear E/D/A/V/R target.

Budget remaining is not a reason to continue.

Agent momentum is not a reason to continue.

A vague feeling that “more review might help” is not a reason to continue.

---

## 10. Failure and retry policy

A retry is justified only when something changes.

Before retrying, identify at least one changed element:
- hypothesis
- input
- command
- dependency
- environment
- implementation
- verification method

If nothing changes, do not expect a different result.

After repeated failure:
1. summarize established facts
2. isolate the unresolved blocker
3. preserve completed work
4. switch route/reasoning tier only if the failure evidence justifies it
5. hand off with state rather than restarting discovery from zero

---

## 11. Routing record

For non-trivial tasks, maintain this compact internal record:

```yaml
route:
  environment: CONVERSATION | WORKSPACE | CODE_AGENT | HYBRID
  reasoning: R0 | R1 | R2
  execution: PROCEED | HANDOFF
  quality_bar: low | normal | high
  key_reason: <one sentence>
  value_target: <E|D|A|V|R + concrete target>
  stop_when: <observable completion condition>
```

Do not expose this record to the user unless:
- a route switch is recommended
- the user asks how the decision was made
- routing itself is the subject of the task

---

## 12. Product adapter: ChatGPT / Work / Codex

Map canonical routes as follows when those surfaces are available:

| Canonical route | Typical OpenAI surface | Typical task |
|---|---|---|
| CONVERSATION | ChatGPT Chat | thinking, writing, judgment, task definition |
| WORKSPACE | ChatGPT Work / persistent workspace | multi-file artifacts, research, long-running project state |
| CODE_AGENT | Codex | repository work, terminal, tests, implementation |
| HYBRID | Chat → Work/Codex | define cheaply, execute deterministically |

Product names and model menus may change. Preserve the canonical logic even if adapters change.

---

## 13. User-interruption policy

Do not interrupt the user with routing details when the current environment is already sufficient.

Interrupt early when:
- current environment is materially more expensive or less reliable
- required execution capability is missing
- continuing here would duplicate work that must later be repeated elsewhere

Keep the warning short and actionable.

Bad:
> I can begin exploring this here and later perhaps you may want to consider moving to another environment...

Good:
> 建议：Codex。需要改仓库并跑测试；先在这里继续会产生重复扫描。把下面这份闭合任务单交给 Codex 即可。

---

## 14. Anti-patterns

Never:
- default every complex-looking task to the strongest model
- default every technical task to a code agent
- use a code agent for unresolved product strategy
- consume a long agent run to discover requirements that could be clarified cheaply
- reload an entire project for a one-file question
- repeat verification with no independent test dimension
- continue because quota remains
- treat token minimization as more important than correctness
- hide uncertainty merely to avoid escalation
- claim background/asynchronous work when none exists

---

## 15. Final invariant

At every stage ask:

> **What new value will the next unit of expensive computation produce?**

If the answer is concrete, proceed.

If the answer is vague, reduce scope, change the method, hand off, or stop.
