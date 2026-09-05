# AI Task Router

**Cost-aware task routing and reasoning governance for AI agents.**

> Route first. Reason second. Stop when marginal value hits zero.

AI Task Router is a portable Agent Skill that decides **where a task should run, how much reasoning it deserves, when to escalate, and when to stop** before expensive execution begins.

It is not a “use fewer tokens” prompt.

Its governing rule is:

> **Every expensive reasoning pass must create new evidence, a new decision, a material artifact delta, a distinct verification result, or measurable risk reduction. Otherwise, stop.**

## Why this exists

Modern AI workflows often waste compute in predictable ways:

- using a code agent before requirements are closed
- scanning the same repository repeatedly
- loading more context than the current step needs
- defaulting to the strongest reasoning mode
- running “one more review” without a new check target
- continuing autonomous loops because budget remains
- doing strategic thinking inside an expensive execution environment

AI Task Router separates **thinking cost** from **execution cost** and routes each stage to the minimum sufficient environment.

## Core model

```text
TASK
  ↓
ROUTE
  ├─ CONVERSATION  → judgment, writing, task definition
  ├─ WORKSPACE     → persistent multi-file / research / artifacts
  ├─ CODE_AGENT    → repo edits, terminal, tests, implementation
  └─ HYBRID        → define cheaply, execute deterministically
  ↓
REASONING BUDGET
  ├─ R0 Direct
  ├─ R1 Standard
  └─ R2 Deep
  ↓
VALUE GATE
  ├─ E: New Evidence
  ├─ D: New Decision
  ├─ A: Artifact Delta
  ├─ V: New Verification
  └─ R: Risk Reduction
  ↓
STOP when the next expensive pass has no concrete value target
```

## OpenAI adapter

| Canonical route | Typical OpenAI surface | Best suited for |
|---|---|---|
| CONVERSATION | ChatGPT Chat | thinking, writing, critique, scope definition |
| WORKSPACE | ChatGPT Work / persistent workspace | multi-file artifacts, research, retained project state |
| CODE_AGENT | Codex | repository work, shell, tests, implementation |
| HYBRID | Chat → Work/Codex | close the spec first, then execute |

The canonical route names are intentionally platform-neutral. Product names and model menus can change without invalidating the routing logic.

## What makes it different

### 1. Minimum sufficient route

Do not ask “what is the strongest model/tool available?”

Ask:

> What is the cheapest environment that can satisfy the quality bar with reliable verification?

### 2. Define before delegate

If execution is expensive and requirements are ambiguous:

1. resolve intent and constraints in conversation
2. write a closed execution brief
3. hand deterministic work to the execution agent

This prevents high-cost agents from burning cycles discovering what they were supposed to build.

### 3. Incremental Value Gate

A new expensive pass is allowed only when it targets at least one new value class:

- **E** — Evidence
- **D** — Decision
- **A** — Artifact change
- **V** — Verification
- **R** — Risk reduction

No target → no pass.

### 4. Orthogonal verification

“Review again” is not a test plan.

Valid:
- correctness review
- security review
- performance review

Invalid:
- comprehensive review
- comprehensive review again
- one more comprehensive review

### 5. Stop is a first-class decision

Remaining quota is not evidence that more work is useful.

The agent stops when the deliverable is usable, acceptance criteria pass, uncertainty is bounded, and the next pass has no distinct E/D/A/V/R target.

## Files

- `SKILL.md` — executable playbook
- `tests/routing-cases.md` — routing regression cases
- `manifest.json` — package metadata
- `LICENSE` — MIT license

## Installation

This project follows the open **Agent Skills / SKILL.md** format. Install or import the `ai-task-router` directory into an Agent Skills-compatible environment.

For systems that support always-on or globally relevant skills, this skill should run at **task intake for non-trivial work**, before high-cost execution begins.

## Suggested usage

You normally should **not** have to invoke it manually.

When installed globally, the agent should silently route tasks when the current environment is sufficient. It should only interrupt you when a different execution surface would materially reduce duplicate work, cost, or failure risk.

Example visible handoff:

```text
建议：Codex + standard reasoning。
原因：需要修改仓库、运行测试并验证结果；继续在 Chat 里做会产生重复扫描。
```

## Design principle

The project optimizes for **marginal value per unit of expensive computation**, not raw token minimization.

That distinction matters: the goal is not to think less. The goal is to stop paying for thinking that produces nothing new.

---

## 中文说明

AI Task Router 是一条面向 AI Agent 的**任务路由与推理成本治理 Skill**。

它在真正开始高成本执行之前，先解决四件事：

1. 这件事应该在哪做？
2. 需要多深的推理？
3. 哪些内容先在低成本环境闭合，再交给 Agent？
4. 什么条件满足后必须停止？

核心原则不是“省 Token”，而是：

> **每一份高成本推理，都必须产生新的信息、判断、交付、验证结果或风险下降。没有新增价值的扫描、复核、重读、改写和 Agent 自循环，都应停止。**

这条 Skill 尤其适合 Chat / Work / Codex 混合工作流，也可以移植到其他支持 Agent Skills 的系统。
