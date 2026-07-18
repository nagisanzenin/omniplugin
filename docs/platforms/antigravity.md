# Google Antigravity (agy)

> Status in engram: **L1 pending — PR #8 open, changes requested 2026-07-18.** External contribution (@mertso13); maintainer re-research shrank the port to ~3 manifest lines. Facts below verified against agy CLI v1.1.4 primary sources.

The most convention-driven loader of the five: a near-empty manifest, whole-repo staging, and directory discovery. If your repo already follows Claude-Code-shaped conventions, you are *almost* AG-native by accident — which is precisely what makes analogy-driven ports dangerous here (they add things AG ignores that other platforms then pick up).

## Loader model

- Install: `agy plugin install <git-url>` — stages the **entire repo** under `~/.gemini/antigravity-cli/plugins/<name>/` (changelog: *"copy the entire plugin directory"*). Bundled `scripts/` ship automatically.
- Manifest: root `plugin.json`, official schema `https://antigravity.google/schemas/v1/plugin.json` — **exactly `name` (required) + `description`, `additionalProperties: false`.** No version, skills, agents, commands fields. (agy tolerates extras today; don't ship them — an optional `version` becomes an ungrepped drift location.)
- Discovery: **directory convention only** — `skills/`, `agents/`, `rules/`, `mcp_config.json`, `hooks.json` at plugin root. Manifest path-mapping isn't part of the format.
- Update: reinstall (no marketplace/update flow found as of v1.1.4).

## Surfaces

| Surface | Mechanism | Notes |
|---|---|---|
| Skills | `skills/` by convention | **auto-derive slash commands** (changelog: "skill-derived slash commands") + intent triggering from frontmatter descriptions — `/learn` `/review` `/coach` exist with zero glue |
| Commands | **no such concept** | not in the structure docs, not in the changelog, absent from the flagship plugin — a `commands/` dir is dead weight here *and* a live hazard on Claude Code ([05 · Pitfalls](../05-pitfalls.md) #2) |
| Subagents | `agents/` convention | frontmatter dialect differs from Claude's (`tools:`/`model:` keys) — spawnability of Claude-format agents **unverified**; if it fails, the caveat is load-bearing when the agent enforces an invariant |
| Hooks | root-level `hooks.json`, **AG's own schema** — not a plugin.json field | events: PreToolUse/PostToolUse/PreInvocation/PostInvocation/Stop (SessionStart only as "legacy"); absolute-path commands; stdin JSON context; `${extensionPath}` resolves to the staged plugin dir. Global: `~/.gemini/config/hooks.json`; workspace: `.agents/hooks.json`. **No session-start equivalent found** → engram ships without the ambient nudge here, and the README says so |
| Root env | **none** | no `CLAUDE_PLUGIN_ROOT` equivalent — root resolution rides the landmark fallback (directory containing `.claude-plugin/plugin.json`), which works *because* staging copies the whole repo |

## The minimal port

```json
{
  "$schema": "https://antigravity.google/schemas/v1/plugin.json",
  "name": "yourplugin",
  "description": "One honest sentence."
}
```

…plus a README callout with the honesty line for whatever didn't port (for engram: "the due-review session nudge isn't ported yet — everything else works the same").

## Review lessons from PR #8 (the whole story is in the PR thread)

1. The contributed `commands/` + fat manifest were **Codex idiom applied by analogy** — inert on AG, regressive on Claude Code. Requested changes *removed* most of the diff.
2. A green `agy plugin validate .` is not evidence: it reported "3 commands" on a platform with no commands concept and "4 skills" (counting `_shared/`). Ask for **runtime** evidence instead: `agy --version`, a `/learn` transcript proving the engine actually shells out, subagent spawnability.
3. Hooks were attempted via a `plugin.json` field and silently skipped — good instinct, wrong door: AG hooks are a root `hooks.json` with their own schema. Port deferred to a follow-up issue rather than bundled half-verified.

## Sources (all cited in the PR review)

- Schema: https://antigravity.google/schemas/v1/plugin.json
- agy CLI changelog: https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md (v1.1.4)
- Flagship reference plugin: https://github.com/EveryInc/compound-engineering-plugin
- Hooks guide: https://medium.com/google-cloud/a-developers-guide-to-agent-hooks-in-antigravity-cli-4c1440febd11
- The PR itself: https://github.com/nagisanzenin/engram/pull/8
