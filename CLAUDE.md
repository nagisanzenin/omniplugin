# omniplugin — agent instructions

This repo is a **playbook**, not a codebase: distilled guidance for building and maintaining omni-platform agent plugins (one repo, one shared core, native installs on many agentic platforms). The reference implementation behind every claim is https://github.com/nagisanzenin/engram — when this playbook and reality disagree, check engram's current main, then fix the playbook.

## Route by task

| You were asked to… | Read, in order |
|---|---|
| Create a **new** omni-platform plugin | `docs/01-anatomy.md` → `docs/02-portability-rules.md`, then the platform files for your launch targets. Copy `templates/session-hook.sh` patterns from day one — retrofitting portability is the expensive way. |
| **Add a platform** to an existing plugin | `docs/03-adding-a-platform.md` (the checklist) + `docs/platforms/<target>.md` if it exists, else start from `templates/platform-intake.md`. Do Phase 1 research from primary sources even when a crib sheet exists — check its date first. |
| **Release / maintain** across platforms | `docs/04-maintenance.md`; engram's `RELEASE_PROTOCOL.md` is the full reference implementation. |
| **Debug** a platform integration | `docs/05-pitfalls.md` (it's probably in there) + the platform crib sheet. |
| Evaluate a **new/unknown platform** | `docs/platforms/landscape.md` for the survey + `templates/platform-intake.md` for the questions. |

## Non-negotiables when applying this playbook

1. Research the target platform's **actual loader** from primary sources before writing glue (schema, CLI changelog, flagship plugin). Never port by analogy.
2. Check every new **root-level path** against every shipped platform's discovery conventions (namespace bleed — pitfall #2).
3. Hooks: self-resolving, silent-unless-useful, degrade to silence, fail closed, `exit 0`.
4. Ship **honest status**: Verified (with the platform version) vs Not-verified (with why it's harmless), in every install doc.
5. The shared core stays byte-identical across platforms. Platform differences live in glue, waterfalls, and install docs — never as forks of core files.

## Maintaining this playbook

- Platform loaders drift monthly. When you learn something new (format change, new gotcha, a claim that stopped being true): update the relevant `docs/platforms/*.md`, refresh its "researched <date>, against <version>" line, and keep the source URL.
- New lesson from shipping a real plugin → add it to `docs/05-pitfalls.md` in the same format: *what happened → the rule*, with the receipt (repo, version, PR/issue number). No rules without receipts.
- New platform gains real support in a real plugin → promote its intake into a full crib sheet in `docs/platforms/`.
- Keep this file's routing table in sync with the actual file tree.
