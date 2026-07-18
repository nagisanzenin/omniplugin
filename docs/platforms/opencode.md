# OpenCode

> Status in engram: **L4 — third platform, shipped v1.0.3 (2026-07-16), hardened v1.0.4.** Contributed by @luanweslley77 (#5, four review rounds; #6, one). The only platform that required *real adapter code*.

OpenCode plugins are npm packages (TypeScript, `@opencode-ai/plugin`), loaded from the npm cache. That one fact drives the entire adapter design.

## Loader model

- Install: `"plugin": ["opencode-engram-learning"]` (npm name) or `"plugin": ["git+https://github.com/<owner>/<repo>.git"]` (pin to source) in `opencode.json` — read globally (`~/.config/opencode/opencode.json`) or per-project.
- **The core problem:** the package lives under `~/.cache/opencode/node_modules/`, and OpenCode does **not** treat the npm cache as a config directory — disk discovery never sees `skills/`/`agents/` inside the package. A naive port is invisible.
- Config dirs that *are* scanned: `~/.config/opencode/` (global) or `{project}/.opencode/` (when the cwd has `opencode.json(c)`).

## Engram's adapter pattern (reusable wholesale)

1. **Self-extract:** the plugin's `config` hook copies `skills/ agents/ scripts/` from the npm cache into the config dir, behind `copyMissing` — **never overwrites existing files**, so user edits survive version bumps. Idempotent via a `.engram-version.jsonc` stamp.
2. **First-execution bridge:** freshly-extracted files aren't discovered until next start, so on first run the hook registers everything through the config layer directly — `cfg.skills.paths.push(...)`, `cfg.command[name] = {template, description}`, agents parsed from frontmatter (custom tools strings → object format, `mode: subagent`, `hidden: true`). Works immediately; disk discovery takes over on the next start and the bridge goes quiet.
3. **Deterministic update flow:** on a version bump, a byte-compare diff (`Buffer.equals`) classifies files; identical → silent, new → extracted, *user-modified* → listed in a manifest + unified `.diff`. A `/engram-update` **pseudo-command** (registered via config only when the manifest exists — never written as a file, so the discovery cache can't lock a stale definition) lets the model show the diff and apply per-file / auto / skip — deletes via `unlinkSync` against a manifest allowlist, an `isWithinTarget` path-traversal guard on every resolved path, zero bash in templates.
4. **Ambient surfaces:** `shell.env` exports `OPENCODE_PLUGIN_ROOT` (the adapter sets the root var *itself* — the platform doesn't); `experimental.chat.system.transform` carries the session nudge; `session.idle` toast announces updates.

## Surfaces

| Surface | Mechanism |
|---|---|
| Manifest | `package.json` (`main` → `.opencode/index.ts`, `files:` allowlist, `engines.opencode`) |
| Skills / agents / commands | extracted copies + config bridge (above) |
| Hooks | plugin SDK hooks: `config`, `shell.env`, `experimental.chat.system.transform`, `session.idle` |
| Root env | `OPENCODE_PLUGIN_ROOT` — self-set, first in the waterfall |
| Distribution | npm (claim the name early — engram holds `opencode-engram-learning` partly against squatting) + git-pin alternative |

## Gotchas

- **SDK config shapes are version-coupled:** the bridge targets v1 shapes (`skills.paths`, singular `command`/`agent`); v2 uses `skills: string[]` and plurals. The adapter documents which it targets and why — pin and note yours.
- **npm publish is a release step** (§6.5 in engram's protocol): skipping it strands npm installers on the old version *and* silently mutes their update notifications (the update manifest diffs against the npm cache). `npm pack --dry-run` first — npm has no gitignore; `__pycache__` will ship. Publishing needs interactive 2FA.
- Adapter code = product code: engram's carries 88 vitest tests + `tsc --noEmit` + engine selftest in CI on every push.

## Sources

- Engram's adapter: https://github.com/nagisanzenin/engram/tree/main/.opencode (heavily commented headers — read `index.ts` and `install.ts` first)
- npm package: https://www.npmjs.com/package/opencode-engram-learning
- OpenCode plugin SDK: https://opencode.ai/docs/plugins (`@opencode-ai/plugin`)
