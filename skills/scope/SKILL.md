---
name: scope
description: Planned scope with an orchestrator session. Use when the user runs /scope, or says a large piece of connected work should be planned as a whole and split into tasks run in separate sessions ("zróbmy to jako scope", "rozpisz to na taski i daj mi prompty do nowych sesji"). Subcommands — plan, dispatch, status, close. Not needed for ordinary task-by-task work; connected tasks are linked automatically by the brief.
argument-hint: "plan <name> | dispatch <TASK> | status | close"
---

# /scope — orchestrate a planned scope

Arguments: `$ARGUMENTS`

This session becomes the **orchestrator**: it holds the big picture, plans and dispatches, and writes no production code. Task sessions do the work and run their own gates. Everything passes through files in `.untracked/decisions/`, so nothing depends on a session remembering anything.

If no subcommand is given: with no scope file for this work yet, run `plan`; otherwise run `status`.

## plan <name>

1. **Gather.** The tickets/description the user gives; any existing notes tagged `Scope: <name>` (the scope may have emerged task by task — adopt those notes and their decisions as the starting point, verified against the code); the relevant code and its dependency radius at the level of the whole flow.
2. **Scope brief** (this replaces the per-task briefs). Working mode: 5–10 sentences on the flow end to end — the parts, how they connect, the cross-task decisions (data model, where logic lives, order of work), what already exists. Learning mode: ask for their naive picture of the flow first, then contrast. Finish with **one or two** forward questions about the flow, and settle the answers before dispatching anything.
3. **Split into tasks.** Use the user's tickets if they exist; otherwise propose a split. For each: key or short name, one-line goal, depends-on. Keep the user's ticket boundaries; suggest changes only when a dependency makes the order impossible.
4. **Write the scope file** `.untracked/decisions/<name>.md`:

```markdown
# Scope: <name>

## Flow
<how the pieces fit, end to end, 5–10 lines>

## Cross-task decisions
- <decision> — because <reason>; applies to <tasks>

## Tasks
| Task | Goal | Depends on | Status |
|---|---|---|---|
| ABC-101 | ... | — | todo |

## Open
<risks, questions to settle later>
```

5. Create each task note (`ABC-101.md`) with `Scope: <name>`, `Brief: done in orchestrator <date>`, `Status: todo`, the goal, the relevant cross-task decisions copied in short, and the assumptions. Then suggest which task to dispatch first.

## dispatch <TASK>

1. Check its dependencies in the task table. If one is not done, say which and ask whether to dispatch anyway.
2. Refresh the task note with anything decided since the plan (later decisions, outcomes of finished tasks it depends on).
3. Print a prompt to paste into a new session in the same repo, in a single code block, short — the files carry the context:

```
<TASK> (scope: <name>). Read .untracked/decisions/<name>.md and .untracked/decisions/<TASK>.md first.
The brief was done in the orchestrator — skip it, except for anything the plan did not foresee.
Goal: <one line>. Depends on: <done tasks and what they provide, one line>.
Do the task, then run the ownership-guard gate before calling it done.
```

If the host offers a tool to start a new session directly, offer to use it instead of copy-paste; the prompt stays the same. Mark the task `in progress` in the table.

## status

Read the scope file and every task note tagged with it. Update the table from the notes (status, one-line outcome), list new decisions from task notes that affect other tasks, flag conflicts between tasks, and say what can be dispatched next. Keep the reply to the table plus a few lines.

## close

When all tasks are done (or the user wants to close it):

1. Run `status` first.
2. **Scope gate**: a summary of the finished flow end to end — which task built which part (`path:line` at the entry points), the cross-task decisions as they ended up — then **1–3 questions** that cross task boundaries: trace a case through the whole flow, the consequence of a cross-task decision, what breaks if one part changes. Learning mode: 3–5, one at a time. Same question rules as the task gates; "skip" is allowed and recorded.
3. Write `## Status` → `closed <date>` and `## Understanding` → result into the scope file; add demonstrated items to `known.md`.

## Switching mid-way

- **Task-by-task work turns out big:** `/scope plan <name>` adopts the existing tagged notes; nothing is lost.
- **The user prefers to stay in one session:** the same scope file works; run each task here with its normal gate, using the scope brief in place of the task briefs.
