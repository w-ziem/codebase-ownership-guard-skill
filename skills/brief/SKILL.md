---
name: brief
description: Ownership-guard pre-build brief. Use when the guard is on (working or learning) and the user hands you a real task — a ticket, feature, bug fix, refactor — before you write or change code. Skip for questions, trivial edits, explicit "skip", or when the approach was already discussed in this session.
---

# Brief — before you build

Purpose: the user knows the direction and the core idea before code appears, so the gate at the end is a confirmation, not a surprise. It must take the user well under a minute in working mode.

## 1. Look before you talk

Do just enough reconnaissance to be concrete: read the ticket/prompt, find where the change will live, compute a first pass of the **dependency radius** (see guard rules), and check `.untracked/ownership/known.md` and any existing `.untracked/decisions/<NAME>.md` for this task (it may be a resumed task — then summarise where it stands instead of re-briefing).

**Dispatched task:** if the note already exists and says `Brief: done in orchestrator`, read it and the scope file it names, then skip the brief. Mention only what the scope plan did not foresee (a surprise in the code, a conflict with a later decision) in two or three lines, and build.

Pick the note `NAME` now (ticket key from prompt or branch, else a short scope-of-work name) and create the note with `Status: in progress`, the `Scope:` tag if there is evidence for one, and the goal/assumptions you are working from. If the ticket is ambiguous, list your assumptions in the brief so the user can correct them in the same reply.

## 2. Find related earlier work

This is how a task-by-task flow keeps its memory without planning. Cheaply:

1. Collect the paths in this task's radius (files you will change plus the ones they rely on).
2. Search `.untracked/decisions/*.md` for those paths and for this task's scope tag (`grep -l`), excluding this task's own note.
3. For each hit, read its `Decisions` and `Status`, and check the relevant code still matches (it may have been changed by someone else since).

Then decide, per relevant earlier decision:
- **Follow it** — this task fits the earlier choice. Say so in one line so the user sees the connection.
- **Propose a change** — this task shows the earlier choice is not enough or conflicts. Say what, why, and what it would touch (`path:line`, which earlier ticket). The user decides; record the outcome as described under "Changed earlier decision" in the guard rules.
- **Note code drift** — the code no longer matches the note. Say so; follow the code, not the note.

If there are no hits, say nothing about it. If an earlier note carries a scope tag and this task clearly belongs to the same flow, adopt that tag.

## 3. Brief — by mode

**Working:**

> **Plan:** <3–5 sentences: the problem, the main idea, the existing pieces it hinges on, where it lives>
> **Related:** <only if any: "follows ABC-101 (status computed in trigger, not flow — bulk)" or "proposes changing ABC-101: …">
> **Assumptions:** <only if any>
> **Check:** <one forward question>

**Learning:** first ask *"How would you do this naively?"* and wait. Then give 4–8 sentences on the intended approach, contrasting it with their naive version and naming related earlier work; define any new concept with a tiny example; finish with one forward question.

## 4. React and go

- Good answer → one line confirming, then build.
- Off-track → two or three sentences fixing the mental model, no second question in working mode; then build.
- "skip" / "dawaj" → build; set `Brief: skipped` in the decision note (otherwise `Brief: done here`).
- A proposed change to an earlier task's decision needs an explicit answer; if the user says skip, follow the earlier decision.
- If their answer reveals a better approach or a requirement you missed, adjust the plan and say so.

Then build without further teaching interruptions in working mode (learning mode: see the build rules). When the work is complete, run `ownership-guard:handover` before you report done.
