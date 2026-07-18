# Hermes Agent (Nous Research)

> Status in engram: **L3 — fourth platform, shipped v1.0.5 (2026-07-18), verified live on Hermes v0.18.2** (macOS, local model; wire-level receipts). Requested in issue #9. [INSTALL-HERMES.md](https://github.com/nagisanzenin/engram/blob/main/INSTALL-HERMES.md).

A multi-surface agent (CLI, TUI, dashboard, Telegram, Discord) with a skills system that reads Agent Skills `SKILL.md` — but no plugin packaging at all. The port is pure configuration.

## Loader model

- **Install route: clone + external dirs.** `git clone` the repo, then in `~/.hermes/config.yaml`: `skills: { external_dirs: [~/repo/skills] }`. Skills join the index, register as `/name` slash commands on every surface, and stay **read-only to the agent** (v0.18.2) — though external dirs are agent-writable via `skill_manage` in some configs; consider a read-only clone.
- **⚠ Never the hub installer.** `hermes skills install` copies each skill folder plus only files referenced *inside it* — "unreferenced repository files are not copied" — which severs shared engines and `_shared/` dirs. This is the canonical referenced-files-only staging hazard (intake question #7).
- Env for the engine: Hermes sets **no plugin-root variable**; `echo "ENGRAM_ROOT=$HOME/repo" >> ~/.hermes/.env` — Hermes loads `.env` at startup and local terminal subprocesses inherit it.
- Bundled Python 3.11 (via uv) — a stdlib-only engine just works.

## Surfaces

| Surface | Mechanism | Verified? |
|---|---|---|
| Skills | `skills.external_dirs` in config.yaml | ✅ live — all skills discovered, `_shared/` correctly ignored, full SKILL.md injected on invocation (sizes measured: 11.5 KB, 17.9 KB) |
| Commands | auto: `/review`, `/coach`; bundles: `~/.hermes/skill-bundles/study.yaml` (`skills: [learn]`) → `/study` — **bundles outrank skills** | ✅ live |
| Ambient hook | top-level `hooks: { pre_llm_call: [{command, timeout}] }` in config.yaml — JSON payload on stdin, `{"context": "…"}` on stdout, `{}` otherwise; consent on first use (`hooks_auto_accept: true` to skip) | ✅ wire-level (request dump inspected) |
| Cron delivery | `hermes cron create --no-agent --script <hook>` — stdout delivered verbatim to Telegram/Discord, **zero LLM cost** | 🟡 recipe-documented, unverified |
| Subagents | `delegate_task(goal, context)` — child starts with "a completely fresh conversation … zero knowledge of the parent's history" → blindness invariants hold structurally; pass the agent prompt file's contents as `context` | 🟡 recipe-documented, not driven end-to-end with a capable model |
| State | on whatever host runs the terminal backend (local/Docker/SSH/Modal/Daytona) | — |

## The collision

`/learn` is Hermes' **own built-in** (its skill-authoring flagship). Hermes detects the clash, skips auto-registering the same-named external skill, and prints the escape hatch: `/skill learn`. Engram's response: document it, ship the optional `/study` bundle, and make the hook rewrite its own nudge text (`sed 's|/learn|/skill learn|g'`) — in **both** hook modes (the cron path initially shipped un-rewritten; a review caught it).

## The dual-mode hook pattern

One script, two protocols, auto-detected by stdin:

- **Hook mode** (stdin carries the `pre_llm_call` JSON): emit `{"context": …}` once per session, `{}` after — dedupe keyed on sanitized `session_id`, **failing closed** to a per-PPID key when extraction fails (at most one nudge per Hermes process, never one per call).
- **Plain mode** (empty stdin — cron `--no-agent`, manual runs): print the rewritten nudge as plain text, no dedupe (each scheduled run *should* deliver), nothing when nothing is due.

Plus the standard contract: self-resolving root, silence on any failure. Shipped with a 9-case failure battery after review found it failing open twice ([05 · Pitfalls](../05-pitfalls.md) #5).

## Gotchas

- Headless `hermes chat -q "…"` passes slash commands through as **literal text** — interactive CLI/TUI/gateway only. Don't let a headless smoke test convince you the install is broken.
- Slash-skill dispatch precedence vs built-ins is undocumented — treat any name shared with a built-in as colliding until proven otherwise.
- Tutoring/behavior quality is the configured model's quality; the mechanics run on a 1B local model, the pedagogy doesn't.

## Sources

- Engram's port: INSTALL-HERMES.md + `hooks/session-start-hermes.sh` (read both; the hook is the reference dual-mode implementation)
- Hermes Agent: https://hermes-agent.nousresearch.com (skills docs; researched at v0.18.2, 2026-07-18)
