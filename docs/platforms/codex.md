# OpenAI Codex CLI

> Status in engram: **L2 with caveats — shipped v1.0.x, built without a live Codex binary; skills-only Route B is the documented fallback.** Researched 2026-07 · [INSTALL-CODEX.md](https://github.com/nagisanzenin/engram/blob/main/INSTALL-CODEX.md).

Codex's plugin, skills, and hooks systems are modeled closely on Claude Code's — most things port 1:1, which is exactly why the two real differences bite.

## Loader model

- Install (plugin route): `codex plugin marketplace add <owner>/<repo>` then `codex plugin add <name>@<marketplace>` (or `/plugin …` in-session).
- Install (skills-only route): any Agent Skills installer — `npx skills add <owner>/<repo>` symlinks skills into agent dirs; Codex's bundled `$skill-installer` also works. Skills land in `~/.agents/skills/<name>/` (legacy `~/.codex/skills/` still read).
- Manifest: `.codex-plugin/plugin.json` — mirrors the Claude manifest but **maps paths explicitly**: `"skills": "./skills/"`, `"hooks": "./hooks/hooks.json"`, plus an `"interface"` block (displayName, category).
- Marketplace catalog: `.agents/plugins/marketplace.json`.

## Surfaces

| Surface | Mechanism | Notes |
|---|---|---|
| Skills | shared `skills/` via manifest mapping | invoked as **`$learn`** (by `$name` mention or the `/skills` picker) — there is no `/learn` slash form; document the spelling |
| Subagents | **TOML**, `codex/agents/*.toml` | `name`, `description`, `model_reasoning_effort`, `sandbox_mode`, `developer_instructions` — and **explicit invocation only**: `$engram-assessor, grade these: …`. No auto-delegation. Plugin-distributed TOML agents aren't a documented feature yet → install via script (`scripts/install-codex.sh` copies to `~/.codex/agents/`) |
| Hooks | same `hooks/hooks.json` shape as Claude Code | whether Codex expands a plugin-root env var inside hooks.json is **unverified** — the hook self-resolves regardless, so a mismatch degrades to silence |
| Root env | `${CODEX_PLUGIN_ROOT}` | second in the resolution waterfall |
| Sandbox | default `workspace-write` | writes **outside** the workspace (e.g. a state dir in `$HOME`) prompt for approval — document state placement + env override |

## The two differences that matter

1. **Subagents are TOML and explicit-only.** Port the prompt verbatim into `developer_instructions`; adapt the *trigger* (the skill's "spawn X" becomes "invoke `$X` with …"); the invariant (e.g. grader blindness) survives unchanged.
2. **The sandbox meets your state dir.** Either keep state inside the workspace, or document the approval prompt and the `ENGRAM_HOME`-style override.

## Honest-status precedent

This port is the template for "shippable without a live binary": verified = shared skills/engine (identical everywhere) + self-resolving hook + TOML ports; not verified = exact marketplace schema, env-var expansion in hooks.json, on-disk cache path — each with why it's harmless and Route B as the fallback. Copy that structure ([template](../../templates/INSTALL-PLATFORM.md)).

## Sources

- Engram's port: https://github.com/nagisanzenin/engram/blob/main/INSTALL-CODEX.md + `.codex-plugin/` + `codex/agents/*.toml`
- Agent Skills standard (the portable layer Codex reads): https://agentskills.io — `SKILL.md` format
