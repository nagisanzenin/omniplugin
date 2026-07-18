# omniplugin

**The playbook for omni-platform agent plugins — one repo, one core, native installs on every agentic platform.**

Agentic platforms are multiplying — Claude Code, OpenAI Codex, OpenCode, Hermes, Antigravity, Gemini CLI, and next quarter's three new ones — and each has its own plugin format, discovery model, hook events, and marketplace. The naive answer is a port per platform, which turns every bug fix into N pull requests. The answer that works is the **omni-repo**: one repository whose portable core (Agent-Skills-standard `skills/` + a zero-dependency CLI engine + one state home) is shared verbatim, wrapped in the thinnest possible per-platform glue.

This playbook is that pattern, extracted from a plugin that actually shipped it: [**engram**](https://github.com/nagisanzenin/engram) runs natively on **Claude Code, OpenAI Codex, OpenCode, and Hermes Agent, with Google Antigravity in review** — same skills, same engine, same user state, on all of them. Every rule in here carries a receipt: the real bug, review round, or user report that taught it. No rules without receipts.

## The doctrine, in ten lines

1. **One portable core, thin glue.** Skills + a deterministic CLI engine, shared byte-identical; platforms get manifests and adapters, never forks.
2. **The shell is the only universal ABI.** The engine is a stdlib-only CLI — every platform can shell out; almost none share a plugin API.
3. **Skills are the portable unit.** Agent Skills `SKILL.md` is the closest thing to a cross-platform standard; write behavior there, once.
4. **One state home.** Learn/work in one tool, continue in another — that's what makes it *omni* rather than N parallel installs.
5. **Resolve roots by waterfall, then landmark.** Env vars where platforms set them; the directory containing your manifest where they don't.
6. **Hooks degrade to silence,** self-resolve, fail closed, and always `exit 0`. An ambient feature must never break a session.
7. **Preserve invariants, not mechanisms.** Your guarantees survive porting; the mechanism enforcing them won't. Map one to the other per platform.
8. **Research loaders, never port by analogy.** Schema, CLI changelog, flagship plugin — before one line of glue.
9. **Check every root path against every platform.** Namespaces overlap; a file added for platform N can change behavior on platform M.
10. **Verified-live or say so.** Every platform doc carries Verified / Not-verified, item by item, forever.

## The map

| Read | For |
|---|---|
| [docs/01-anatomy.md](docs/01-anatomy.md) | The omni-repo shape: core vs glue, the five-platform comparison table, the L0–L4 integration ladder |
| [docs/02-portability-rules.md](docs/02-portability-rules.md) | The ten engineering rules that make one repo run everywhere — each with code and its receipt |
| [docs/03-adding-a-platform.md](docs/03-adding-a-platform.md) | **The checklist** — the recurring operation, phase by phase, with the 12 intake questions |
| [docs/04-maintenance.md](docs/04-maintenance.md) | Operating the matrix: the release protocol pattern, version-grep, honest status, community as your test fleet |
| [docs/05-pitfalls.md](docs/05-pitfalls.md) | Field notes: eleven real failures → eleven rules |
| [docs/platforms/](docs/platforms/) | Crib sheets: [Claude Code](docs/platforms/claude-code.md) · [Codex](docs/platforms/codex.md) · [OpenCode](docs/platforms/opencode.md) · [Hermes](docs/platforms/hermes.md) · [Antigravity](docs/platforms/antigravity.md) · [the wider landscape](docs/platforms/landscape.md) |
| [templates/](templates/) | [platform-intake.md](templates/platform-intake.md) (the 12 questions) · [INSTALL-PLATFORM.md](templates/INSTALL-PLATFORM.md) (install-doc skeleton) · [session-hook.sh](templates/session-hook.sh) (the reference ambient hook) |

Working with an agent? Point it at [CLAUDE.md](CLAUDE.md) — it routes by task.

## The case study in one table

engram v1.0.5, 2026-07-18 — what "one core, five platforms" concretely costs and buys:

| Platform | Glue that was needed | Rung |
|---|---|---|
| Claude Code | 2 JSON manifests (the conventions the core is written in) | L4 |
| OpenAI Codex | mirrored manifest + 3 TOML agent ports + install script | L2 (unverified against live binary; honest-status'd) |
| OpenCode | the one real adapter: TS self-extract + config bridge + deterministic updater (88 tests) | L4 |
| Hermes Agent | zero repo changes for skills (config entries) + one dual-mode hook variant | L3 (verified live, wire-level) |
| Antigravity | a 3-line manifest — discovery does the rest | L1, in review |

Shared and byte-identical across all five: `skills/`, `agents/*.md` (as prompt source), `scripts/engram.py` (214 selftest checks, same count everywhere), one state dir.

## Scope, honestly

This is distilled from **one plugin shipping five platforms** (plus a researched survey of the rest of the landscape). It is a playbook, not a spec: platform loaders drift monthly, so every platform fact here is dated and versioned, and [CLAUDE.md](CLAUDE.md) tells future maintainers how to keep it true. When this playbook and a platform's current docs disagree, the platform docs win — then fix the playbook.

## More from the same workshop

- **[engram](https://github.com/nagisanzenin/engram)** — the reference omni-plugin: evidence-based learning with blind-graded recall and FSRS scheduling, on five platforms.
- **[effortmining](https://github.com/nagisanzenin/effortmining)** · **[idiolect](https://github.com/nagisanzenin/idiolect)** · **[production-grade](https://github.com/nagisanzenin/claude-code-production-grade-plugin)** · **[less](https://github.com/nagisanzenin/less)** — Claude Code plugins sharing the same habit: deterministic cores, receipts, and honest numbers.

---

<sub>MIT · Built by extracting what shipping actually taught, so the next plugin starts from rules instead of re-earning them.</sub>
