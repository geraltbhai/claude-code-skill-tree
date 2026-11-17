---
name: context-audit
description: Finds what fills a Claude Code session's context and which repo files an agent should never read, then proposes Read deny rules, CLAUDE.md and MCP changes for approval. Use when sessions feel slow or expensive, usage limits hit early, or before working in an unfamiliar repo.
---

# Context audit

Every request re-sends the whole context: CLAUDE.md, files read, tool results, messages. Smaller context means cheaper, faster, more focused sessions. This skill measures first, proposes changes second, and changes nothing until the user approves.

```
Context audit:
- [ ] 1. Measure the session (user runs /context and /usage)
- [ ] 2. Scan the repo for files an agent can't use
- [ ] 3. Classify candidates; drop anything the agent needs
- [ ] 4. Draft Read deny rules with the tokens each one saves
- [ ] 5. Check CLAUDE.md and MCP overhead
- [ ] 6. Present the report; wait for approval
- [ ] 7. Apply, then confirm the rules loaded and re-measure
```

## 1. Measure the session

Built-in slash commands are typed by the user, so ask for their output:

- `/context`: what fills the window now (memory files, MCP tools, messages).
- `/usage`: session tokens and cost, the prompt-cache line, and on Pro, Max, Team and Enterprise plans a breakdown by skill, subagent, plugin and MCP server with recommendations.

If the user skips this step, continue with the repo scan and say the session numbers are missing.

## 2. Scan the repo

Prefer ctxray. It is offline and has no dependencies:

```bash
npx -y github:geraltbhai/ctxray            # breakdown by file and category
npx -y github:geraltbhai/ctxray --json     # machine-readable, includes wasteCategories
```

Without Node, fall back to the largest tracked files:

```bash
git ls-files -z | xargs -0 wc -c 2>/dev/null | grep -v " total$" | sort -n | tail -40
```

Estimate tokens as bytes / 4 when ctxray isn't available, and say it's an estimate.

## 3. Classify

Flag these, if they're large:

| Category | Typical files |
|---|---|
| Lockfiles | `package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`, `uv.lock`, `poetry.lock`, `Cargo.lock` |
| Build output | `dist/`, `build/`, `.next/`, `coverage/`, `out/` |
| Minified or bundled | `*.min.js`, `*.bundle.js`, source maps |
| Generated code | `*.gen.ts`, `*_pb2.py`, `routeTree.gen.ts`, schema dumps |
| Snapshots and fixtures | `__snapshots__/`, `tests/fixtures/`, recorded HTTP responses |
| Bulk data | large `.json`, `.csv`, `.txt` corpora, SVG or drawio diagrams |
| Vendored | `vendor/`, `third_party/` |

Never flag:

- Source directories, even ones named `packages/`, `lib/`, `bin/` or `src/`. Monorepos and CLIs keep real code there.
- `.d.ts` type declarations, which are high-value context.
- Config files the agent edits: `package.json`, `tsconfig.json`, CI files.
- Docs the user maintains, and anything the current task touches.

When unsure, leave the file out of the proposal and list it under "worth a look".

## 4. Draft Read deny rules

`.claudeignore` has no effect in Claude Code. The supported mechanism is a `Read` deny rule in `.claude/settings.json` (shared with the team) or `.claude/settings.local.json` (personal).

Rules use gitignore syntax:

- A bare filename such as `package-lock.json` matches at any depth.
- A single-directory pattern in a deny rule, such as `dist/**`, also matches nested copies.
- A path with several segments, such as `tests/fixtures/**`, is anchored at the project root.

```json
{
  "permissions": {
    "deny": [
      "Read(package-lock.json)",
      "Read(dist/**)",
      "Read(tests/__snapshots__/**)"
    ]
  }
}
```

If ctxray already wrote an `.agentignore`, convert it:

```bash
grep -vE '^[[:space:]]*(#|$)' .agentignore \
  | sed -E 's#/$#/**#; s#.*#Read(&)#' \
  | jq -R . | jq -s '{permissions:{deny:.}}'
```

Tell the user the limits plainly. Claude Code applies `Read` rules to its file tools, Grep, Glob and @-mentions on a best-effort basis, and to shell commands it recognises, such as `cat` and `head`. The rules don't stop `grep -r` run from a parent directory, or a script that opens files itself. The sandbox covers that.

## 5. CLAUDE.md and MCP

- Count lines in every CLAUDE.md and CLAUDE.local.md that loads at launch. Anthropic's guidance is under 200 lines per file. If over, recommend the `claude-md-slim` skill.
- From `/context`, list MCP servers whose tools take a noticeable share. Suggest disabling unused ones in `/mcp`, and preferring a CLI such as `gh` when it does the job.

## 6. Report

```
## Context audit · <repo>

Session: <from /context, or "not provided">
Repo: <total tokens> total · <flagged %> flagged

| Proposed deny rule | Why | Tokens saved |
|---|---|---|
| Read(pnpm-lock.yaml) | lockfile, 8 copies | 813k |

CLAUDE.md: <lines> lines (target under 200)
MCP: <servers worth disabling, or "nothing notable">
Worth a look (not proposed): <files you weren't sure about>

Nothing has been changed yet. Apply these?
```

## 7. Apply and verify

- Merge into the existing settings file. Never overwrite other keys; deny lists from different settings files combine.
- Ask the user to run `/status` (the "Setting sources" line confirms the file loaded) and `/permissions` to see the rules.
- Re-run ctxray. It reads `.agentignore` back, so the flagged total should drop.
