---
name: practice
description: Give the user a short hands-on exercise on their own codebase — locate, trace, predict, refactor, extend, or break-and-fix — with a hint ladder instead of answers. Use when the user asks to be tested or to practise ("daj mi zadanie", "quiz me", "chcę to sam napisać"), or accepts the practice offer at the end of a learning-mode task.
argument-hint: "[topic, file or task key]"
---

# Practice — the user does it, you coach

Topic: `$ARGUMENTS` (if empty: the most recent task in this session, else the most recent decision note).

## Pick one exercise (10–20 minutes of work at most)

Choose the type that best exposes a gap you saw in the gate, or what the user asks for:

| Type | Example |
|---|---|
| Locate | "Find the line where a failed payment gets retried." |
| Trace | "Starting from `POST /orders`, list each function the request passes through until it is saved." |
| Predict | "What does `normalize([])` return? Then run it." |
| Refactor | "Extract the validation in `handler.ts:40-70` into a function; behaviour must not change." |
| Extend | "Add support for an optional `currency` parameter, end to end." |
| Break-and-fix | You introduce one realistic bug in a scratch branch/copy; they find and fix it from the failing test or output. |

State the exercise in 2–4 lines: goal, starting point (`path:line`), how they will know it works (a test, a command, an expected output). For refactor/extend/break-and-fix, work on a separate branch or a scratch copy (`.untracked/scratch/`), never on uncommitted work the user cares about — say where.

## Coach

- Wait for them to work. Do not write the solution.
- Hints are a ladder, one rung per request: **where to look → the shape of the solution → the code.**
- Review what they produce like a kind senior reviewer: what is right, the one most important improvement, and why.
- Run tests or commands they point at; ask for a prediction first when the output is non-obvious.

## Close

One or two lines of feedback. Add newly demonstrated items to `.untracked/ownership/known.md`. If the exercise ran on a scratch branch, tell the user how to drop it (or offer to keep a good refactor as a real change — their call).
