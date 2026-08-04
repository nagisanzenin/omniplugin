# OpenClaw

> Status in engram: **L3 — sixth platform, shipped v1.10.x, verified live on OpenClaw 2026.7.1-2** (macOS, isolated state dir, live model). The first platform that is a **chat gateway rather than a coding tool** — the agent sits behind Discord, Telegram, WhatsApp, iMessage, Signal — which is the case that proved the omni-repo pattern generalises past the terminal. `/review` from a phone is what an FSRS schedule was always waiting for.
>
> Researched 2026-07-22, against OpenClaw 2026.7.1-2.

OpenClaw ships **no plugin format of its own that you'd target** — it reads *other platforms'* manifests as "bundles". That single fact produces most of this page: the port needed zero new manifests, and every subtlety lives in how OpenClaw reinterprets files you shipped for someone else.

## Loader model

- Install: `openclaw plugins install <name> --marketplace <owner>/<repo>` — reads the **Claude** `marketplace.json` straight from GitHub. A local clone path works too (`openclaw plugins install /path/to/repo`, the track-main route).
- Staging: `${OPENCLAW_STATE_DIR:-~/.openclaw}/extensions/<name>/`, full repo — so the landmark fallback and shared docs come along.
- **Bundle detection precedence disagrees with its own docs** (pitfall #12): the docs say a native `openclaw.plugin.json` wins; the shipped code checks `.codex-plugin/plugin.json` **first**. Ship a Codex manifest and you are a Codex bundle here forever. Engram deliberately does **not** ship a native manifest — inert today, it would silently change the plugin's whole shape the day upstream fixes the precedence.
- Skills: bundle skill roots load as native skills (symlinked into `plugin-skills/`), joined to the skill index — `/learn` works on every chat surface, and natural language activates skills by description.
- No root env var → resolve at `${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/extensions/<name>` in the waterfall.

## The surfaces that matter

| Surface | Mechanism |
|---|---|
| Skills / commands | via the borrowed bundle manifest; slash commands on every connected chat surface |
| Ambient nudge | **hook pack** `hooks/<name>/` (`HOOK.md` + `handler.js`) on `command:new` / `command:reset` — the only two internal events whose output routes back to the originating conversation, i.e. the honest SessionStart equivalent |
| Global gate | `openclaw config set hooks.internal.enabled true` — **without it, zero plugin hooks run** while `openclaw hooks list` still shows `✓ ready` (pitfalls #14–15). This is an install *step*, not troubleshooting |
| Subagents | none registered — no bundle format maps `agents/` (Claude-format bundles detect the dir and explicitly don't execute it). Use `sessions_spawn` with its default `context: "isolated"`: a clean child transcript, which is the isolation invariant by construction (R6). Non-blocking — `sessions_yield`, result arrives as the next message |
| State | on the **gateway host** (learn at your desk, review from your phone — same store); if the gateway is a VPS, that host owns the state |

## The manifest trap, spelled out

For Codex bundles, OpenClaw treats the manifest's `hooks` value as **a list of directories to scan for hook packs**. Engram's Codex manifest said `"hooks": "./hooks/hooks.json"` — a file path, correct on Codex — so OpenClaw scanned a JSON *file* for `*/HOOK.md`, found nothing, loaded nothing, and still listed `hooks` as a bundle capability. The fix was **deleting the key**: Codex auto-discovers `./hooks/hooks.json` by convention, OpenClaw then scans `./hooks/` where the pack lives, and both platforms find their hooks by being told nothing (pitfall #13). When platforms share a manifest they share a namespace, not a schema.

## sessions_spawn and tool policy

`sessions_spawn` sits behind tool policy: the `coding` and `full` profiles include it; `messaging` and `minimal` do not — and without it there is no isolated child, so an invariant that rides it must **stop and say so** rather than degrade (engram: no blind grader → no receipts; the install doc names `tools.profile: "coding"` / `tools.alsoAllow`). Pass work to the child by **file path, never inline** — a task string is both a shell-injection and prompt-injection surface.

## Verified live (2026.7.1-2) — and not

**Verified:** all three install routes; Codex-bundle detection with skills+hooks capabilities; skills discovered and a live model named them unprompted; `/review` and `/coach` driven end-to-end against a seeded store; the hook observed registering (`Registered hook: … -> command:new, command:reset` — zero handlers without the flag, six with it) and pushing the nudge onto `event.messages`; a full `sessions_spawn` round-trip in which the child ran in its own session, read the agent file from the installed plugin, returned a valid receipt — **and the child's submitted prompt was inspected in the trajectory log: task text only, no parent transcript.** A deliberate negative: pointed at a missing instructions file, the child returned `{"status":"blocked"}` and refused to improvise.

**Not verified:** delivery of the nudge to a *real connected chat surface* (the handler demonstrably pushes the text; the CLI agent path has no originating conversation to reply into, so it drops); a complete `/learn` session; the artifact smith — whose HTML-file output over a chat channel is an open design question, not a solved one.

## Sources

- engram's port: [INSTALL-OPENCLAW.md](https://github.com/nagisanzenin/engram/blob/main/INSTALL-OPENCLAW.md) (the manifest-trap maintainers note and itemized honest status) · [`hooks/engram-due/`](https://github.com/nagisanzenin/engram/tree/main/hooks/engram-due) · `skills/_shared/subagents.md` (the isolated-spawn contract)
- OpenClaw docs: https://docs.openclaw.ai — bundles, plugins, internal hooks; for anything load-bearing, read the loader's code (pitfall #12 exists because the docs and code disagree)
