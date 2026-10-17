---
name: recall
description: Answer "where is X from ABC-123", "what did we do in <task>", "why is Y done like that", "what did I work on yesterday / this week" from the ownership-guard decision notes, verified against the current code. Use when the user asks about past work in this repo, prepares a standup, review or handover to a colleague.
argument-hint: "<ticket key, scope, or question>"
---

# Recall — answer from the notes, verify in the code

Question: `$ARGUMENTS`

1. **Find the notes.** List `.untracked/decisions/`. Match by ticket key or scope name; otherwise search the notes' text for the topic. For "yesterday / this week", use file modification dates and the `Status` dates.
2. **Verify against today's code.** Notes record what was true when written. For every `path:line` you are about to cite, check it still exists and still does that (code moves; use git log / search to find where it went). Say when something changed since the note ("moved to `x.ts:120` in commit abc123 by someone else").
3. **Answer short**, in the user's language, in a form they could say out loud to a colleague:
   - where it is (`path:line`, clickable),
   - what it does and why it was done that way (from **Decisions**),
   - status and anything open.
4. If there is no note, say so plainly, then answer from the code and git history (`git log -S`, `git log --grep=<KEY>`) and offer to create the note.

For a standup/summary request: one line per task — key, status, what moved, blocker if any.

Recall is not a quiz — never ask questions here unless the user asks to be tested.
