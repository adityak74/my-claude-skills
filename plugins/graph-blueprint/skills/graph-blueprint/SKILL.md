---
name: graph-blueprint
description: Use when a question or task is broad enough that one pass will miss things - research, audits, reviews, comparisons, migrations, "be thorough" requests - and the work can be split across parallel agents whose results must be verified in fresh context and merged into a single report.
---

# Graph Blueprint

## Core principle

Run the work as a graph: **nodes do the work, edges carry the results.**

```text
GOAL -> SPLIT -> [WORKER 1..N in parallel] -> VERIFIER -> MERGE -> FINAL ANSWER
                  research / compare / check / find gaps
```

Six stages, in order:

| # | Stage | Node does |
|---|---|---|
| 1 | Goal | define what you want, in one testable sentence |
| 2 | Split | break the work into independent pieces |
| 3 | Workers | fan out: many agents, different angles, in parallel |
| 4 | Verifier | reduce: check quality in **fresh context**, catch mistakes and hallucinations |
| 5 | Merge | combine everything that survived verification |
| 6 | Final answer | synthesize one clear, actionable report |

Use the goal supplied after `/graph-blueprint` as stage 1. Skip the whole pattern when the task is small, tightly sequential, or already answered by a single file read — a graph costs several agent runs and is waste on trivial work.

## Non-negotiable rules

- **Never let a worker verify its own output.** The verifier gets a fresh context and only the claim, not the worker's reasoning.
- **Never skip the verifier** because workers "seemed confident". Confidence is exactly what the verifier exists to test.
- **Workers get different angles, not the same prompt N times.** Redundancy without diversity finds redundant results.
- Each worker prompt must be self-contained: goal, scope, angle, output shape. Workers cannot see each other.
- **Never merge unverified findings** into the final answer, and never present a refuted finding as a caveat instead of dropping it.
- Report what was dropped and what was not covered. Silent truncation reads as complete coverage.
- Never spawn workers that write to the same files in parallel without isolation.
- Escalate to the user instead of guessing when the goal itself is ambiguous — a graph built on the wrong goal wastes every node.

## 1. Goal

Write the goal as one sentence stating the deliverable and its success test.

- Bad: "look into the auth code"
- Good: "list every place session tokens can outlive logout, with file:line and a concrete failure case for each"

If the goal has more than one deliverable, either pick one or run one graph per deliverable. Do not let a worker discover the goal.

## 2. Split

Split by **angle**, not by alphabet. Good splits are independent (a worker never needs another worker's output) and jointly exhaustive.

| Split axis | Use when |
|---|---|
| By source | evidence lives in distinct places (code, docs, tests, git history, web) |
| By dimension | one artifact, several lenses (correctness, security, performance, UX) |
| By hypothesis | competing explanations to confirm or kill in parallel |
| By region | large surface with natural partitions (services, packages, directories) |

Four canonical worker roles from the blueprint, useful when no better split exists: **research** (gather), **compare** (weigh alternatives), **check** (validate claims against reality), **find gaps** (what's missing — a modality not run, a source unread, a claim unverified).

Announce the split before running it:

```text
Worker  Angle              Scope                 Returns
W1      research           src/auth/**           findings[] {claim, file:line, evidence}
W2      compare            W1-free: prior art    options[] {approach, tradeoff}
W3      check              tests + CI history    claims[] {status, proof}
W4      find gaps          whole repo            gaps[]  {missing, why it matters}
```

If two workers would need to talk to each other, the split is wrong — merge them into one node or sequence them as two stages.

## 3. Workers (fan out)

Dispatch all workers for a stage **in one message** so they run concurrently. Each prompt states:

1. the goal sentence,
2. this worker's angle and scope boundary,
3. the exact output shape (fields, one item per finding),
4. that its final message *is* the return value — no preamble, no narration.

Require evidence per item: `file:line`, a command and its output, or a URL. **A finding with no evidence is discarded at merge, not verified** — say so in the prompt.

Prefer read-only workers. When workers must edit, give each an isolated worktree or disjoint file set.

Do not wait idle: keep doing main-thread work that no worker depends on while they run.

## 4. Verifier (reduce)

Fresh context is the point. Give the verifier the claim and its evidence — never the worker's chain of reasoning, which is what carries the hallucination.

Prompt the verifier to **refute**, not to review: *"Try to refute this claim. Default to refuted when the evidence does not independently support it."*

| Findings | Verification depth |
|---|---|
| Few, low stakes | one refuter per finding |
| Many / high stakes | 3 refuters per finding, majority kills it |
| Multi-mode risk | one refuter per lens (correctness, security, reproducibility) |

Verdicts: **confirmed** (evidence checks out), **refuted** (drop it), **unsupported** (plausible, no proof — report separately, never as a finding).

## 5. Merge

Mechanical, not creative. Do it yourself, not in an agent:

1. Drop refuted and evidence-free items.
2. Deduplicate by `(file, line, claim)` — the same issue found by three angles is one item, with the strongest evidence kept.
3. Rank by severity or by impact on the goal.
4. Keep a dropped-items tally: how many refuted, how many unsupported, what went uncovered.

If merging leaves nothing, that is a valid result. Report "no confirmed findings" — never pad with unverified material.

## 6. Final answer

One report, written for someone who never saw the graph:

```text
Answer:     <direct response to the goal sentence>
Findings:   <ranked, each with evidence and why it matters>
Unsupported:<plausible but unproven, clearly labeled>
Coverage:   <what was searched, what was not, what was dropped and why>
Next:       <concrete actions, or none>
```

Never present the process as the deliverable. Worker counts and stage timings belong in one line at most.

## Loop until dry (optional)

For unknown-size discovery (bugs, gaps, edge cases), repeat stages 3–5 until **two consecutive rounds surface nothing new**. Deduplicate each round against everything *seen*, not just against confirmed items — otherwise refuted findings reappear every round and the loop never converges.

## Common mistakes

| Mistake | Fix |
|---|---|
| Worker verifies its own finding | Separate node, fresh context, refute-first prompt |
| Same prompt to N workers | Split by angle; diversity is the reason to fan out |
| Barrier between every stage | Only synchronize when a stage genuinely needs all prior results |
| Findings without evidence | Require `file:line` / command output / URL in the worker contract |
| Graph on a trivial task | One read beats six agents; skip the pattern |
| Merge as another agent | Dedup and ranking are deterministic — do them directly |
| Report describes the graph | Report answers the goal; process is a footnote |

## Red flags

Stop and fix the graph when you are about to:

- send a worker's own output back to that worker for checking;
- write the final report while any finding is still unverified;
- launch workers before the goal sentence is written down;
- fan out with prompts that differ only in file paths;
- present a refuted finding "for completeness";
- claim full coverage when a planned angle was skipped or truncated.
