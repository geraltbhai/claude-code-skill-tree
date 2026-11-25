# Claude Code Skill Tree

Ten Claude Code skills in three tiers that build on each other. First stop paying for context nobody needs, then tighten the edit-test loop, then run agents without burning tokens.

**See the tree:** https://geraltbhai.github.io/claude-code-skill-tree/

| Tier | Skills | |
|---|---|---|
| I · Lean Context | `context-audit`, `claude-md-slim`, `session-handoff` | **Free, MIT, in this repo** |
| II · Fast Loops | `test-output-filter`, `model-router`, `plan-gate` | Full tree |
| III · Agent Leverage | `agent-team-lead`, `independent-verifier`, `skill-scout` | Full tree |
| Capstone | `usage-review` | Full tree |

## Install Tier I

```bash
git clone --depth 1 https://github.com/geraltbhai/claude-code-skill-tree
cp -r claude-code-skill-tree/skills/* ~/.claude/skills/        # every project
# or, for one project only:
cp -r claude-code-skill-tree/skills/* .claude/skills/
```

Then ask Claude Code in plain words, for example "run a context audit", "slim down my CLAUDE.md" or "write a handoff, I'm switching tasks". Claude loads a skill when the request matches its description.

## What Tier I does

**`context-audit`** measures what fills your session (from `/context` and `/usage`), scans the repo with [ctxray](https://github.com/geraltbhai/ctxray) for lockfiles, snapshots, generated code and bulk data, and proposes `Read` deny rules with the tokens each one saves. It knows `.claudeignore` has no effect in Claude Code and says what deny rules don't cover.

**`claude-md-slim`** gets CLAUDE.md under about 200 lines by moving each section to a home that actually loads on demand: skills for workflows, path-scoped rules for one area of the code, docs for reference material. It won't move text into `@imports`, which load together with CLAUDE.md and save nothing.

**`session-handoff`** writes a 40-line HANDOFF.md and tells you when to `/clear` (free) instead of `/compact` (a large request in itself), so you don't pay to re-send a stale session or eat a full-price cache miss after a break.

Every skill that changes settings, CLAUDE.md or hooks shows the diff and the expected saving first, and applies nothing without a yes.

## Grounding

Mechanisms were checked against the Claude Code docs on 17 Nov 2026: [costs](https://code.claude.com/docs/en/costs), [permissions](https://code.claude.com/docs/en/permissions), [memory](https://code.claude.com/docs/en/memory), [settings](https://code.claude.com/docs/en/settings). If Claude Code changes and a skill is out of date, please open an issue.

## Full tree

Tiers II, III and the capstone are in the full tree: [get it on Gumroad](https://samyakvaghela.gumroad.com/l/claude-code-skill-tree) ($29 launch price). New to all this? Start with the [$9 Claude Code Playbook kit](https://samyakvaghela.gumroad.com/l/claude-code-playbook): 7 prompts plus the hook, agents and templates they use.

---

By [EigenSys](https://eigensys-website.vercel.app/), makers of [ctxray](https://geraltbhai.github.io/ctxray/). Not affiliated with Anthropic.
