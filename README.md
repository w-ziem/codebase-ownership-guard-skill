# ownership-guard

**Delegate the whole ticket to Claude. Still know what was done, where, and why.**

A Claude Code plugin that runs in every session as a light "vibe-coding gate". Claude works at full speed, and two short touchpoints per task make sure you own the result:

1. **Brief** before Claude builds: the core idea in a few sentences, plus one question to check you've got the direction.
2. **Gate** before the task counts as done: what changed and where (`file:line`), which existing code it relies on, the key decisions, and 1–3 questions you answer in the session. Say "skip" whenever you need to. The skip gets recorded honestly.

Along the way it keeps a **decision note** per ticket or scope in `.untracked/decisions/<TICKET>.md`. When someone asks "hey, where's X from ABC-123?", you look it up there (or run `/recall ABC-123`) instead of asking Claude.

Works for any stack, because nothing in it is language- or framework-specific: Salesforce, data science, backend, AI engineering.

## Two modes

| | **working** (default) | **learning** |
|---|---|---|
| Goal | Know the process, the architecture of the change, where things live and the key technical decisions | Understand the code as if you wrote it |
| Before | 3–5 sentence plan + 1 question | "How would you do it naively?" → approach vs. naive → 1 question |
| During | Uninterrupted. Code stays production-clean: no comments for you, no `TODO(you)` | Interactive. At most one `TODO(you)` piece per task for you to write, with a hint ladder; predict-before-run |
| Gate | 1–3 architecture/process questions | 3–5 questions one at a time: locate the line, trace a call, predict, naive vs. actual |
| After | Decision note | Decision note + optional exercise (`/practice`) |

### Dependency radius

You don't only need to understand the diff. You need the diff **plus the existing code it calls**: the service methods your controller uses, and what those call in turn, down to the framework/library boundary. In other words, exactly what you would have read if you had written the change by hand, minus the searching. If a code-graph tool is available in the session (graphify, an LSP, a repo index MCP), Claude uses it to compute the radius. Otherwise it falls back to targeted search.

`.untracked/ownership/known.md` remembers what you've already demonstrated, so after six months in the same repo you get asked about what's new or changed, not the same service for the tenth time.

## Install

Inside a Claude Code session (terminal app):

```
/plugin marketplace add w-ziem/codebase-ownership-guard-skill
/plugin install ownership-guard@ownership-guard
```

Pick the **user** scope ("for you, across all projects") so the guard is on in every repo. Restart the session or run `/reload-plugins`.

From a local clone (for hacking on it):

```
/plugin marketplace add /path/to/ownership-guard
/plugin install ownership-guard@ownership-guard
```

To try it without installing: `claude --plugin-dir /path/to/ownership-guard`.

**Requirements:** bash. On Windows, Git for Windows (which Claude Code already needs) is enough.

### Make sure `.untracked/` is never committed

Add it once to your global git excludes:

```bash
git config --global core.excludesFile   # shows the path, if one is set
echo ".untracked/" >> ~/.config/git/ignore   # default location when none is set
```

The plugin checks this before it writes anything and asks you if `.untracked/` isn't ignored.

## Switching modes

| Command | Effect |
|---|---|
| `/guard status` | Show the active mode and where it comes from |
| `/guard working` | Working mode, globally |
| `/guard learning` | Learning mode, globally |
| `/guard off` | Disable the guard (no briefs, gates or notes) |
| `/guard learning repo` | Learning mode only in the current repo |
| `/guard repo clear` | Remove the repo override |

Or just say it: "switch to learning", "turn the guard off for this session". A session-only change writes nothing to disk.

The mode is resolved in this order, highest first:

1. env var `OWNERSHIP_GUARD_MODE` (e.g. in a project's `.claude/settings.json` → `"env"`)
2. `<repo>/.untracked/guard-mode`
3. `~/.claude/ownership-guard/mode`
4. default `working`

Skill names are namespaced by the plugin (`/ownership-guard:guard`). The short `/guard` works when no other skill has the same name.

**Turning it off completely:** `/guard off` keeps the plugin installed but silent. To remove it, run `/plugin disable ownership-guard@ownership-guard` (or `uninstall`).

## Skills

| Skill | When |
|---|---|
| `guard` | Switch or show the mode |
| `brief` | Automatically, before Claude builds a real task |
| `handover` | Automatically, before Claude reports a task as done, and when a task gets blocked or interrupted (records the state so you can resume) |
| `recall` | "Where is X from ABC-123?", "what did we do in case-management?", "what did I do this week?", standup prep. Answers from the notes, verified against current code |
| `scope` | Opt-in orchestration of a big piece of connected work: `plan`, `dispatch <TASK>` (prints a prompt for a new session), `status`, `close` |
| `practice` | A hands-on exercise on your own code: locate, trace, predict, refactor, extend, break-and-fix |

What's exempt: questions, exploration, trivial edits, and anything you say "skip" to. If you already discussed the approach with Claude in the session (brainstorm, plan mode), the brief counts as done.

## Connected tasks

Tasks are often links in one flow: prerequisites of each other, the same area, the same epic. There are two ways to handle that, and both write the same notes.

**Task by task (default, nothing to do).** You keep working one ticket at a time, with no planning up front. Every brief looks for earlier notes that touch the same files as this task's dependency radius, or that share its `Scope:` tag. When it finds one, it either follows the earlier decision ("follows ABC-101: status is computed in the trigger, not a flow, because of bulk") or proposes changing it and names what that would touch. A changed decision gets recorded in both notes, so `/recall ABC-101` stays truthful. The `Scope:` tag fills itself in from the epic, an earlier note, or the area you mention, and `/recall <scope>` uses it to assemble the whole flow.

**Planned scope (`/scope`).** Use this when you know up front that it's one big piece and you want an orchestrator session:

1. `/scope plan case-management`: one brief for the whole flow, cross-task decisions, a split into tasks, and the scope file `.untracked/decisions/case-management.md` with a task table.
2. `/scope dispatch ABC-102`: refreshes the task note and prints a short prompt to paste into a new session. The files carry the context, so the prompt only points at them. The task session skips its own brief (it was done in the orchestrator) and runs the normal task gate.
3. `/scope status`: rebuilds the task table from the task notes and flags conflicts.
4. `/scope close`: the scope gate, with questions that cross task boundaries.

If task-by-task work turns out big, `/scope plan` adopts the notes that are already tagged. The same scope file also works if you stay in one session.

## How it works

- `hooks/guard` is a small bash script run on **SessionStart**. It injects the rules for the active mode (`modes/common.md` plus `modes/working.md` or `modes/learning.md`).
- On **UserPromptSubmit** it adds a one-line reminder, so the rules survive long sessions and mode switches apply immediately.
- It never blocks a tool call. If the script fails, the session continues without the guard.

## Decision note format

```markdown
# ABC-123 — <title>
Scope: case-management   ← or none
Brief: done here | done in orchestrator | skipped
## Status
## Goal & assumptions
## What changed          ← path:line + what/why
## Relies on             ← dependency radius
## Decisions             ← choice, reason, rejected alternative; [changed in ABC-104] entries
## Open / next           ← how to resume
## Understanding         ← passed | partially explained | skipped by user
```

## Credits

The question design builds on ideas from [Altitude](https://github.com/jasonku09/altitude-skills) by Jason Ku: free recall instead of multiple choice, questions that probe forward instead of backward, evidence-based "known" tracking, and the hint ladder. Altitude is a full learn-to-code tutor. This plugin targets people who already code and want to keep owning AI-written work.

## License

MIT
