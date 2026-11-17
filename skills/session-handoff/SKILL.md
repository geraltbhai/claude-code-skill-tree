---
name: session-handoff
description: Ends a long Claude Code session cheaply by writing a short HANDOFF.md, then tells the user whether to /clear or /compact and how to resume. Use when switching to unrelated work, before a long break, when context is getting full, or when the user says "wrap up", "hand off" or "start fresh".
---

# Session handoff

A long session re-sends its whole history with every request. Two moments make that expensive:

- **Unrelated work in the same session.** Every new message still carries the old task's context.
- **A break longer than the prompt-cache lifetime.** The first message afterwards misses the cache and pays full price for the entire context.

`/clear` starts fresh and costs nothing. `/compact` keeps continuity but has to read the whole conversation to summarise it, so it is a large request in itself. A written handoff gives most of the continuity of `/compact` at the price of `/clear`.

```
Handoff:
- [ ] 1. Decide: clear, compact, or keep going
- [ ] 2. Write HANDOFF.md (40 lines max)
- [ ] 3. Give the user the exact commands
```

## 1. Decide

| Situation | Do |
|---|---|
| Next task is unrelated | Handoff, then `/clear` |
| Stepping away for a while, same task after | Handoff, then `/clear`; resume from the file |
| Same task, deep in a tricky debug with lots of live detail | `/compact <what to keep>` |
| Short session, same task, no break | Keep going |

## 2. Write HANDOFF.md

Write it at the repo root (or the path the user prefers). Facts only, no narration:

```markdown
# Handoff · <date> · <task in one line>

## Goal
<what "done" means, one or two lines>

## Done
- <change> (<file>)

## Next
1. <next concrete step>
2. ...

## Decisions
- <decision> because <reason>

## Watch out
- <gotcha, flaky test, env quirk>

## Useful commands
- `<test or run command>`
```

Keep it under 40 lines. Link to files instead of pasting code. If the repo ignores scratch files, suggest adding `HANDOFF.md` to `.gitignore` the first time.

## 3. Tell the user

For a fresh start:

```
1. /rename <short-name>    (so you can /resume this session later if needed)
2. /clear
3. First message next time: "Read HANDOFF.md and continue from Next."
```

For continuity in the same session:

```
/compact Keep: the goal, open steps, decisions and reasons, files changed, failing tests. Drop: exploration that led nowhere.
```

Never run `/clear` or `/compact` for the user, and never delete the handoff file without asking.
