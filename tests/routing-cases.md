# AI Task Router — Routing Regression Cases

These cases are not benchmark scores. They are behavioral regression tests for the routing policy.

A compatible implementation should reach the expected route unless platform capability or user constraints materially change the situation.

## Case 01 — Rewrite one short email

**Input**  
Rewrite a 150-word email to sound concise and professional.

**Expected**
- Route: `CONVERSATION`
- Reasoning: `R0`
- Execution: `PROCEED`
- Value target: `A` — improved final text
- Stop: usable rewrite delivered

**Must not**
- route to CODE_AGENT
- perform repeated style reviews without a named defect

---

## Case 02 — Decide product positioning before implementation

**Input**  
A user has three competing product directions and wants to choose one before building.

**Expected**
- Route: `CONVERSATION`
- Reasoning: `R1` or `R2` depending on consequence/ambiguity
- Execution: `PROCEED`
- Value target: `D` — close the strategic choice

**Must not**
- send an implementation agent to prototype all three by default

---

## Case 03 — Implement a repository feature

**Input**  
Add OAuth login to an existing repository, update tests, run the suite, and fix failures.

**Expected**
- Route: `CODE_AGENT`
- Reasoning: `R1`
- Execution: `HANDOFF` if currently in plain conversation
- Value targets: `A + V`
- Stop: implementation complete and required tests/checks pass

---

## Case 04 — Ambiguous feature plus repository work

**Input**  
“Add a smarter search experience to my app.” No acceptance criteria are defined.

**Expected**
- Route: `HYBRID`
- Stage 1: `CONVERSATION`, R1, close scope and acceptance criteria
- Stage 2: `CODE_AGENT`, R1, deterministic implementation and verification

**Must not**
- let CODE_AGENT discover product requirements through repeated code changes

---

## Case 05 — Multi-document research report

**Input**  
Synthesize 20 documents into a report, maintain citations, revise findings, and preserve project state for later updates.

**Expected**
- Route: `WORKSPACE`
- Reasoning: `R1`
- Value targets: `E + A + V`
- Stop: report satisfies required structure/citations and unresolved uncertainty is bounded

---

## Case 06 — One-file factual question inside a large repo

**Input**  
Explain what a known configuration value in one specified file does. No edits are requested.

**Expected**
- Route: `CONVERSATION` if the file content is already available; otherwise a minimal repository/file read capability
- Reasoning: `R0` or `R1`

**Must not**
- scan the entire repository

---

## Case 07 — Failed test retry

**Input**  
A test failed. The agent already reran the identical test once with identical code and environment.

**Expected**
- Do not rerun again until at least one of hypothesis/input/implementation/environment/verification method changes.
- Value target before retry: `E`, `A`, or `V` must be concrete.

**Must not**
- repeat the same action solely because failure persists

---

## Case 08 — “Review it again” after a full review

**Input**  
A complete correctness review has already passed. The user says “review again” without naming another concern.

**Expected**
- Ask internally whether a distinct verification dimension exists.
- If none exists, stop or state that another identical pass is unlikely to add value.
- If risk justifies more verification, choose an orthogonal dimension such as security, performance, citations, or consistency.

**Must not**
- perform an identical comprehensive review loop

---

## Case 09 — High-consequence architectural decision

**Input**  
Choose a data architecture that affects migration risk, privacy, latency, and long-term operating cost.

**Expected**
- Route: `CONVERSATION` for architecture decision; possibly `HYBRID` for later implementation
- Reasoning: `R2`
- Value targets: `E + D + R`
- Stop: trade-offs are explicit, critical unknowns are resolved/bounded, decision is made

---

## Case 10 — Mechanical batch rename across a repo

**Input**  
Rename a symbol across 80 files and run tests.

**Expected**
- Route: `CODE_AGENT`
- Reasoning: `R0` or `R1`
- Value targets: `A + V`

**Principle tested**
A large operation is not automatically a deep-reasoning problem.

---

## Case 11 — Large remaining quota after task completion

**Input**  
The deliverable is complete, acceptance criteria pass, tests pass, and no unresolved issue remains. Significant quota is still available.

**Expected**
- Stop.

**Must not**
- invent new review loops merely because budget remains

---

## Case 12 — Current environment is sufficient

**Input**  
A normal reasoning task is correctly handled in the current conversation environment.

**Expected**
- Route silently.
- Do not show routing metadata to the user.

**Principle tested**
Routing governance should reduce friction, not create ceremony.

---

# Pass criteria

The skill passes this regression set when it consistently demonstrates all five invariants:

1. **Minimum sufficient route** — strongest environment is not the default.
2. **Minimum sufficient reasoning** — deep reasoning is justified, not habitual.
3. **Define before delegate** — ambiguity is closed before expensive deterministic execution when practical.
4. **Incremental Value Gate** — repeated expensive passes require a distinct E/D/A/V/R target.
5. **Explicit stop** — completion ends the loop even when compute budget remains.
