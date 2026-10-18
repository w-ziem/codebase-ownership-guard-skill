---
name: handover
description: Ownership-guard gate before a task counts as done. Use when the guard is on and you have finished (or are about to report finishing) a real task — before saying "done", committing, or opening a PR. Summarises what changed and where, checks the user's understanding with a few questions in the session, then writes the decision note. Also use when a task gets blocked or interrupted, to record its state.
---

# Handover — the gate before "done"

The task is **not done** until the user passes this gate or explicitly skips it. Do not say "done", "finished", "ready to merge" or open a PR before that. Committing locally is fine if the user asked for it.

## If the task is blocked or interrupted (not finished)

Skip the questions. Update `.untracked/decisions/<NAME>.md`: `Status: blocked by <X>` or `paused`, exactly what is done, what is half-done (files, branch, uncommitted state), and the next step to resume. Tell the user in two lines. Done.

## 1. Prepare (silently)

- `git diff` / `git status` against where the task started (or the files you changed in this session).
- Learning mode: `git grep -n "TODO(you)"` — any hit means the task is not finished; deal with it first.
- Finalise the **dependency radius** for what actually got built.
- Read `.untracked/ownership/known.md`; drop anything already owned from the questions.

## 2. Present the summary

Compact, scannable, no prose padding:

```
**ABC-123 — <title>**
What & where
- `src/x/service.ts:88` — <what + why>
- ...
Relies on
- `Repo.findActive` (src/x/repo.ts) — filters soft-deleted rows; we depend on that for <...>
Decisions
- <decision> — <reason> (not <alternative>: <reason>)
```

Use clickable `path:line` references. Working mode: architecture-level. Learning mode: include the non-obvious lines.

## 3. Ask

- **Working:** 1–3 questions in one message, process/architecture level.
- **Learning:** 3–5 questions, one at a time (locate, trace, predict, naive-vs-actual, change).

Pick questions where a wrong answer would hurt them later: the decision a reviewer will challenge, the dependency with a side effect, the failure mode. Follow the common question rules (free recall, forward, grade substance, max two rounds per point).

## 4. Close

When the answers are in (or the user says skip):

1. Update the decision note: `Status: done — <date>`, what changed, relies on, decisions, open/next, and `Understanding: passed | partially explained | skipped by user` with one line on what was covered. No questions or answers in the note.
2. If this task changed an earlier task's decision, add the `[changed in <KEY>, <date>]` line to that older note. If the note's scope has a scope file (planned scope), update this task's line in its task list (status + one-line outcome) and add any cross-task decision to it.
3. Append to `.untracked/ownership/known.md` only what the user demonstrated in their own words (create the file with a `# Known` heading if missing).
4. One closing line: the task is done, where the note is (in a dispatched session: tell the user they can return to the orchestrator). In learning mode, add the one-line practice offer.
