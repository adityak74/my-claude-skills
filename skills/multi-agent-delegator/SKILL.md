---
name: multi-agent-delegator
description: Use when a coding goal can be split across visible cmux panes and delegated to Codex or Antigravity, especially for parallel implementation, testing, exploration, documentation, or independent review while the main agent retains coordination and integration.
---

# Multi-Agent Delegator

## Core principle

Act as the lead engineer in the current Claude Code pane. Delegate bounded work to interactive Codex and Antigravity sessions in visible cmux panes; retain architecture, coordination, review, and integration.

Use the goal supplied after `/multi-agent-delegator` as the objective. Do not delegate when the work is tiny, tightly sequential, or safer to complete directly.

## Non-negotiable rules

- Always launch delegates in a **new visible cmux pane**.
- Start the agent interactively with exactly `codex --yolo` or `agy --bypass-dangerous-permissions`.
- **Never pass the task as a CLI argument.**
- Send the task through cmux to the delegate's explicit surface, then send Enter as a separate action.
- Never rely on the focused pane for cross-pane actions; always target `--surface`.
- Never let writable parallel delegates share a working directory when overlap is possible.
- Never authorize credentials, production changes, force-push/history rewrite, user-data deletion, or irreversible external actions unless the user explicitly approves that exact action.
- Never auto-integrate work that is untested, out of scope, conflicted, or security-sensitive.
- Leave delegate panes and worktrees open unless the user requests cleanup.

## 1. Inspect

Before routing:

1. Read repository instructions such as `CLAUDE.md`, `AGENTS.md`, and `GEMINI.md`.
2. Check repo root, branch, status, test commands, and whether the goal depends on uncommitted changes.
3. Load an installed cmux skill when available. Its verified syntax overrides examples here.
4. Verify tools rather than inventing flags:

```bash
command -v cmux codex agy
cmux ping
cmux identify --json
cmux capabilities --json
```

If cmux, the chosen delegate, explicit surface targeting, or screen reading is unavailable, stop and report the missing capability.

## 2. Route

Announce a compact plan, then proceed unless the user overrides it:

```text
Agent          Objective                 Mode       Validation
Codex-1        Implement parser          Worktree   pytest tests/parser -q
Antigravity-1  Add edge-case tests       Worktree   pytest tests/parser -q
Main           Coordinate/review/merge   Current    full test suite
```

Prefer:

| Delegate | Best fit |
|---|---|
| Codex | implementation, refactoring, bug repair, build/test fixes, objective acceptance criteria |
| Antigravity | exploration, test authoring, documentation, independent approach, broad repository analysis |
| Main agent | architecture, cross-task decisions, sensitive judgment, review, integration |

Default to at most three delegates. Duplicate a task across agents only when explicitly comparing approaches is worth the cost.

## 3. Isolate

Use a dedicated Git worktree when any of these is true:

- more than one writable delegate runs;
- file overlap is possible or uncertain;
- current branch is `main`, `master`, a release branch, protected, or shared;
- the task is a broad refactor;
- semantic conflict risk is material.

Create a unique branch and worktree from committed state, for example:

```bash
git worktree add -b "delegate/<agent>-<task>" \
  "$HOME/.claude/worktrees/<repo>/<run>/<agent>-<task>" HEAD
```

Shell-quote every generated path. A shared directory is allowed only for read-only work or one clearly disjoint writable delegate on a non-protected branch.

If the goal depends on dirty, uncommitted changes and safe isolation cannot preserve them, ask the user. Do not silently commit, stash, discard, or copy those changes.

## 4. Launch through cmux

For each delegate:

1. Snapshot surfaces with `cmux list-panels --json --id-format refs`.
2. Create a split with `cmux new-split right` or `cmux new-split down`.
3. List surfaces again and record the newly created `surface:N`.
4. Send only the startup command to that surface, then Enter:

```bash
cmux send --surface "$SURFACE" -- "cd -- <shell-quoted-workdir> && exec codex --yolo"
cmux send-key --surface "$SURFACE" enter
```

For Antigravity, replace the startup command with:

```bash
cd -- <shell-quoted-workdir> && exec agy --bypass-dangerous-permissions
```

5. Read the pane until the interactive prompt is visibly ready:

```bash
cmux read-screen --surface "$SURFACE" --lines 40
```

Retry startup once after inspecting the pane. A second failure marks the task blocked.

## 5. Deliver the task

Every delegate contract must state:

- objective and acceptance criteria;
- assigned working directory and branch;
- allowed scope and likely files/subsystem;
- repository instructions and constraints;
- required validation commands;
- no merge, cherry-pick, force-push, deployment, credential access, or edits outside scope;
- ask before destructive, external, production, security-sensitive, or ambiguous actions;
- commit only scoped changes after tests pass when using a worktree;
- completion format shown below.

For a short one-line contract:

```bash
cmux send --surface "$SURFACE" -- "$TASK_PROMPT"
cmux send-key --surface "$SURFACE" enter
```

Do not inject raw multiline text into an interactive TUI. For a longer contract, write it to a temporary file under `/tmp/multi-agent-delegator/<run>/<task>.md`, then send a one-line instruction through cmux:

```bash
cmux send --surface "$SURFACE" -- "Read <shell-quoted-prompt-file>, execute that contract, and report using its completion format."
cmux send-key --surface "$SURFACE" enter
```

The delegate must end with:

```text
DELEGATION_COMPLETE
Summary: <what changed or found>
Files: <paths>
Tests: <commands and results>
Commit: <hash or none>
Risks: <none or details>
```

Use `DELEGATION_BLOCKED` with the blocking question or failure when it cannot finish.

## 6. Monitor and steer

Track each worker in the main pane with states `STARTING`, `WORKING`, `WAITING`, `BLOCKED`, `COMPLETE`, or `FAILED`.

At natural checkpoints, inspect the exact surface:

```bash
cmux read-screen --surface "$SURFACE" --lines 60
```

Do not busy-loop. Continue main-agent work between checks. When a worker asks:

- answer routine, scope-preserving questions through `cmux send`, followed by a separate Enter;
- drive a TUI menu only after reading the current screen, using `cmux send-key --surface ...`;
- escalate architecture changes, ambiguous trade-offs, credentials, destructive actions, production access, or external side effects to the user.

A stalled or looping worker receives one concise corrective prompt. If it still makes no progress, mark it blocked; do not nudge indefinitely.

## 7. Review and integrate

For every completed writable task:

1. Read the completion report and inspect `git status`, commits, and the full diff from the task base.
2. Reject unrelated edits, secrets, generated junk, destructive changes, or violations of repository instructions.
3. Run the delegate's validation independently when practical.
4. Confirm the destination branch is clean and has no unresolved semantic conflict.
5. Auto-integrate only when all checks pass. Prefer cherry-picking the reviewed delegate commit, then rerun relevant tests on the integrated branch.
6. If any gate fails, leave the work isolated and report the exact issue. Resolve only trivial, well-understood conflicts; escalate semantic conflicts.

Minor local defects may be fixed directly and revalidated. Material defects go back to the same delegate or remain isolated.

## 8. Final report

Report:

```text
Agent | Surface | Worktree/branch | Task | Result | Tests | Integration
```

Then summarize completed, blocked, and failed work; commits integrated; remaining decisions; and the panes/worktrees left open for inspection.

## Red flags

Stop and correct the orchestration when you are about to:

- run `codex --yolo "<task>"` or `agy --bypass-dangerous-permissions "<task>"`;
- send to whichever pane happens to be focused;
- paste a multiline contract directly into the TUI;
- launch writable delegates on a protected branch or overlapping shared directory;
- treat bypass flags as authorization for risky actions;
- integrate based only on a delegate's claim that tests passed;
- close panes or remove worktrees before the user can inspect them.
