---
name: guard
description: Switch or show the ownership-guard mode (working, learning, off), globally or for the current repo. Use when the user runs /guard, or says things like "switch to learning mode", "turn the guard off", "włącz tryb learning", "wyłącz guarda", "jaki mam tryb".
argument-hint: "working | learning | off | status  [repo]"
---

# /guard — switch mode

Arguments: `$ARGUMENTS`

Modes: `working`, `learning`, `off`. Scope: global by default; add `repo` to set it only for the current repository.

Where the mode lives (highest wins):
1. env var `OWNERSHIP_GUARD_MODE` (e.g. set in a project's `.claude/settings.json` → `env`)
2. `<repo root>/.untracked/guard-mode` — repo override
3. `~/.claude/ownership-guard/mode` — global
4. default: `working`

## Steps

**`status` or no argument:** run the plugin's hook in `mode` form and report the effective mode and where it comes from, in one or two lines:

```bash
"${BASH:-bash}" "${CLAUDE_PLUGIN_ROOT}/hooks/guard" mode
```

If `CLAUDE_PLUGIN_ROOT` is not set in your shell, read the files from the list above directly instead.

**`working` / `learning` / `off` (global):**

```bash
mkdir -p ~/.claude/ownership-guard && printf '%s\n' <mode> > ~/.claude/ownership-guard/mode
```

**With `repo`:** write the same value to `.untracked/guard-mode` in the repo root (create `.untracked/` if needed, and first check it is git-ignored as described in the guard rules). **`repo clear`** deletes that file so the global mode applies again.

If an env var or repo override would shadow the value you just set globally, say so.

## After switching

The switch applies immediately: from now on in this session follow the new mode's rules (summarised below), and every new session starts in it. Confirm in one line, e.g. "Guard: learning (global)."

- **working** — light brief before building; gate with architecture-level questions before done; no learner artefacts in code.
- **learning** — naive-version brief; interactive build with at most one `TODO(you)` piece and a hint ladder; deeper gate (locate / trace / predict); optional exercise.
- **off** — no briefs, gates or decision notes until switched back on. If you switch mid-task to off, finish the task without a gate but still write the decision note if one already exists for it.

For a one-session change without touching files ("only for this session"), do not write anything — just follow the requested mode until the session ends and say that the stored mode is unchanged.
