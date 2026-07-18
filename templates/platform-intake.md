# Platform intake — <PLATFORM> (researched <DATE>, against <VERSION>)

> Answer all 12 **in writing, from primary sources** (official schema, CLI changelog, one flagship plugin), before writing any glue. "Modeled on Claude Code" is not an answer — see the engram PR #8 story for what analogy ships.
> When done, distill this into `docs/platforms/<platform>.md` and keep the source URLs.

## 1 · Manifest
Required file & location: ·
Schema URL: ·
Fields that are real vs merely tolerated: ·
Does it want a `version` field? (If optional → omit; it's a drift location.)

## 2 · Discovery model
Manifest-mapped or directory convention? ·
Which root paths are scanned, exactly: ·

## 3 · Skills
Reads Agent Skills `SKILL.md` natively? ·
Skills auto-derive commands? ·
Frontmatter dialect differences: ·

## 4 · Agents / subagents
Format (md/toml/yaml) + frontmatter keys: ·
**Trigger semantics: auto-delegated or explicit-only?** ·
Can plugins distribute agents, or separate install? ·

## 5 · Hooks
Lifecycle events available (is there a session-start?): ·
Protocol (exit code / stdout / JSON-in-JSON-out): ·
Registered where (manifest / hooks file / user config)? ·
Per-session or per-LLM-call firing? (per-call → dedupe, fail closed)

## 6 · Environment
Plugin-root env var set by the platform: ·
What reaches subprocess shells (and how do users add vars)? ·

## 7 · Staging model ⚠ the load-bearing one
Whole repo, or referenced-files-only? ·
Staged where on disk: ·
Does shared core (engine, `_shared/`) survive the platform's own installer? ·

## 8 · Update flow
How users get new versions: ·
Any caches that go stale (and the workaround): ·

## 9 · Sandbox / permissions
Can hooks/skills write outside the workspace? ·
Will the state dir trigger prompts? ·

## 10 · Namespace collisions
Platform built-ins that collide with our names: ·
Platform behavior on collision + escape hatch: ·

## 11 · Headless vs interactive
Do slash commands work non-interactively? ·
How NOT to test the install: ·

## 12 · Distribution & demand
Marketplace / registry / git-url / manual? ·
Who asked for this port (issue #): ·
Who can verify live, if we can't: ·

---

**Invariant mapping** (from [02 · Portability rules](../docs/02-portability-rules.md) R6): for each plugin invariant, the mechanism here — or the documented degradation:

| Invariant | Mechanism on this platform | Verified? |
|---|---|---|
| · | · | · |

**Target rung** (L0–L4) and what's deliberately deferred: ·
