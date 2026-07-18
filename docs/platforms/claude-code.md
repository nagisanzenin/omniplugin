# Claude Code (Anthropic)

> Status in engram: **born here — L4, fully native.** Researched against live use, continuously. Last updated 2026-07-18.

The richest plugin surface of any agentic platform, and the de-facto template the others imitate (Codex's system is "modeled closely" on it). Treat Claude Code as your source-of-truth format; port outward from it.

## Loader model

- Install: `claude plugin marketplace add <owner>/<repo>` + `claude plugin install <name>@<marketplace>`. Stages the full repo in the plugin cache.
- Discovery: **manifest + convention.** `.claude-plugin/plugin.json` is the manifest; `skills/`, `agents/`, `commands/`, `hooks/` at root are auto-discovered with no manifest declaration.
- Update: `claude plugin marketplace update <name>` then `claude plugin update <name>@<marketplace>`, then restart or `/reload-plugins`. ⚠ A plain `plugin update` *before* the marketplace refresh reports "already current" against the stale cache — put both commands, in order, in your release notes.

## Surfaces

| Surface | Mechanism | Notes |
|---|---|---|
| Manifest | `.claude-plugin/plugin.json` | name, version, description, author, homepage, keywords |
| Marketplace | `.claude-plugin/marketplace.json` | lets the repo *be* its own marketplace (`source: "./"`) — zero-infrastructure distribution |
| Skills | `skills/*/SKILL.md` | auto-discovered; frontmatter `name`/`description` drive both `/command` and intent triggering |
| Commands | root `commands/*.md` | auto-discovered **with no manifest field** — ⚠ namespace-bleed hazard for files added for other platforms. If a skill and a command share a name, **the skill takes precedence** (the command becomes dead weight) |
| Subagents | `agents/*.md` | markdown + frontmatter (`tools:`, `model:`); **auto-delegated** — "MUST BE USED …" descriptions genuinely route |
| Hooks | `hooks/hooks.json` | events incl. SessionStart (matcher `startup\|resume\|clear`), PreToolUse, etc.; command hooks with timeout |
| Root env | `${CLAUDE_PLUGIN_ROOT}` | set for hooks and expanded in hooks.json commands; available to skill shell snippets |
| MCP | `.mcp.json` / manifest | full support |

## What engram ships for it

`.claude-plugin/{plugin.json,marketplace.json}` + the shared `skills/`, `agents/`, `hooks/hooks.json` → SessionStart nudge. No adapter code at all — this platform defines the conventions the core is written in.

## Gotchas

- **Skill-over-command precedence** silently kills same-named commands — and root `commands/` auto-discovery means another platform's files can leak in as live commands (the engram PR #8 near-miss: [05 · Pitfalls](../05-pitfalls.md) #1–2).
- The marketplace cache staleness two-step (above) is the #1 "update didn't work" support question.
- SessionStart hooks fire on `startup|resume|clear` — output is injected as context; a noisy hook pollutes every session. Silent-unless-useful is the contract.

## Sources

- Plugins reference: https://code.claude.com/docs/en/plugins-reference.md ("Skills and commands are automatically discovered when the plugin is installed")
- Skills: https://code.claude.com/docs/en/skills.md (same-name precedence)
- Living example: https://github.com/nagisanzenin/engram (`.claude-plugin/`, `hooks/`)
