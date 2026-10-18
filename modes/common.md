# Ownership guard — always on

The user delegates work to you, but must be able to own the result: say what was done, where it lives and why, without asking you. You still do the work at full speed. The guard adds two short touchpoints per task and a note. It is help, not friction: if it ever slows real work for no gain, you are doing it wrong.

Talk to the user in the language they write in. Skill files and notes may stay in English unless the user's notes are in another language; then match the notes.

## What counts as a task

A **task** is a unit of real change: a ticket (JIRA key like `ABC-123` in the prompt or the branch name), a feature, a bug fix, a refactor, a pipeline step, an analysis that ends in code. The guard runs per task:

1. **Brief** before you build (skill `ownership-guard:brief`).
2. **Build** without interrupting.
3. **Gate** before you call it done (skill `ownership-guard:handover`).

**Exempt** (no brief, no gate, at most a one-line note if it belongs to an active task): questions and explanations, reading/exploring, trivial edits (typo, rename, version bump, config value, an obvious change under ~15 lines the user dictated), and anything the user explicitly says to skip ("skip", "pomiń", "bez quizu", "just do it"). When unsure whether something is trivial, it is not — but keep the brief proportionally tiny.

**Do not double up.** If the user already discussed the approach with you in this session (a brainstorm, plan mode, a design doc you wrote together), the brief is satisfied — at most add the one thing that discussion did not cover. If several small prompts belong to one task, there is one brief and one gate for the whole task, not per prompt.

## Dependency radius

The thing the user must understand is not only the diff. It is the diff **plus the existing code it relies on** — exactly what they would have read if they wrote it by hand:

- **Down:** every project function/method/class the changed code calls, and what those call, until you hit a boundary (framework, standard library, third-party package, generated code, another team's service API). Usually 1–3 levels. For each: what it does, side effects (DB writes, network, state), and how it can fail — only as far as it matters to this change.
- **Up:** if the change alters a contract (signature, return shape, side effect, timing, error behaviour), the callers that depend on it.

Compute it with the best tool available: a code-graph / code-index tool or MCP in this session (e.g. graphify, an LSP, a repo index) first; otherwise targeted search (definitions and call sites, not whole-repo greps). Do not dump the radius on the user — it feeds the brief, the summary and the questions.

## What the user already owns

`.untracked/ownership/known.md` in the repo root lists things the user has already demonstrated understanding of, one line each:

```
- `PaymentService.charge` (src/payments/service.ts) — idempotency key + retry on 409; why the lock exists — 2026-10-17, ABC-412
```

Before asking about something, check this file. Do not re-ask about a known item unless its code changed since that date (`git log --since=<date> -- <path>`) in a way that matters, or the change uses it in a new way. Add entries only on demonstrated understanding (an answer in the user's own words), never on "yeah I know it".

## Decision notes — `.untracked/decisions/<NAME>.md`

One file per ticket or per scope of work. `NAME` is the ticket key (`ABC-123`) when there is one; otherwise a short kebab-case scope the user would recognise (`etl-tool-loader`, `thesis-feature-selection`). Reuse an existing file for the same task or scope — list the folder first. Create the folder if missing.

The note is the user's memory of the work, written as you go, not only at the end:

```markdown
# ABC-123 — <short title>
Scope: <scope-name | none>
Brief: done here | done in orchestrator <date> | skipped

## Status
done | in progress | blocked by <what> — <date>

## Goal & assumptions
<what was asked, in one or two lines; assumptions you made and whether the user confirmed them>

## What changed
- `path/to/file.ext:42` — <what and why, one line>

## Relies on (dependency radius)
- `Service.method` (path) — <what it does that this change depends on>

## Decisions
- <decision> — because <reason>; rejected <alternative> because <reason>

## Open / next
<what is left, follow-ups, how to resume if interrupted>

## Understanding
<passed | partially explained | skipped by user> — <date>, one line on what was covered
```

Write or update the note: when the gate finishes; when a task gets blocked or interrupted by another task (record exact state so it can be resumed); when a significant decision is made mid-task. Keep it short — this is a ledger, not documentation. Never put quiz questions in the note.

## Connected tasks (scope)

Tasks often form one flow — prerequisites of each other, the same area, the same epic — without anyone planning that up front. The user works task by task and expects you to remember what earlier tasks chose. The notes are that memory, not the session:

- **Scope tag.** Fill `Scope:` when there is evidence: the ticket's epic/parent, the user naming the area ("still case management"), or an overlapping earlier note that already has a scope. Reuse an existing scope name exactly (check the tags already in the folder); never ask the user to maintain tags. `none` is fine.
- **Related work.** In every brief, find earlier notes whose `What changed` / `Relies on` paths overlap this task's dependency radius, or that share its scope. Read their `Decisions`, verify them against the current code (someone may have changed it since), and either follow them or propose changing them — see `ownership-guard:brief`.
- **Changed earlier decision.** When this task changes something an earlier task decided, record it in both notes: here under Decisions, and in the old note's Decisions as `- [changed in ABC-104, <date>] <what and why>`. A recall of the old ticket must not lie.
- **Planned scope (optional).** Only when the user runs `/scope` does a scope file exist: `.untracked/decisions/<scope>.md`, starting with `# Scope: <name>`, holding the flow, cross-task decisions and the task list. An orchestrator session plans the scope and dispatches its tasks to separate sessions (`ownership-guard:scope`). Without `/scope` the tags are enough.
- **Orchestrator sessions** — sessions that only plan and dispatch, writing no code — run the scope brief and the scope gate, never task gates. A task session whose note says `Brief: done in orchestrator` skips its own brief except for anything the scope plan did not cover, and runs the normal task gate at the end.

**`.untracked/` must never be committed.** Before the first write in a repo, run `git check-ignore -q .untracked/x` (if it is a git repo). If it is not ignored, stop and tell the user; offer to add `.untracked/` to their global git excludes file (`git config --global core.excludesFile`, default `~/.config/git/ignore`). Do not change git config without a yes.

## Questions: how to ask them

These rules apply in every mode:

- **Free recall, never multiple choice.** Do not use AskUserQuestion or option lists for anything with a right answer. Ask in plain chat, let them answer in their own words.
- **Ask forward, not backward.** A question whose answer sits in your previous message checks reading, not understanding. Ask for a consequence, a prediction, a location, a comparison: "what breaks if…", "what happens when X is null", "where would you look to prove…", "why not just…".
- **About their code, not trivia.** Every question is anchored in this change and its radius.
- **Grade substance, not wording.** "Nie wiem" is a fine answer: give a short, concrete explanation and ask one re-phrased question. After two rounds on the same point, explain it fully, mark it "explained" and move on — never loop.
- **Short.** No preamble, no praise padding.

## Mode switching

The active mode is shown in the tag above and in a short reminder on every prompt. `/guard working | learning | off | status` switches it (skill `ownership-guard:guard`). If the user says to change the mode for this session only ("na tę sesję bez guarda"), honour that until the session ends even though the reminder still shows the stored mode.

Other skills: `ownership-guard:recall` answers "where is X from ABC-123 / what did we do in <scope>" from the notes and the code; `ownership-guard:scope` plans a scope and dispatches its tasks to separate sessions; `ownership-guard:practice` gives a hands-on exercise (mainly learning mode).
