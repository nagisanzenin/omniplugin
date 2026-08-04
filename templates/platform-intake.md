# Platform intake — <PLATFORM> (researched <DATE>, against <VERSION>)

> Answer all 15 **in writing, from primary sources** (official schema, CLI changelog, one flagship plugin), before writing any glue. "Modeled on Claude Code" is not an answer — see the engram PR #8 story for what analogy ships.
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

## 13 · Foreign-manifest compatibility
Does this platform read another platform's manifest (a "bundle")? Which formats? ·
If we ship several, **which marker wins** — and is that answer from the docs or the loader's code? ·
Any key in that borrowed manifest that means something **different** here? ·

## 14 · Global gates
Per capability we contribute: does the host require a separate opt-in before it runs? ·
The exact config key and the command that sets it (this becomes an install *step*): ·

## 15 · What the diagnostics actually prove
Does `list` / `inspect` / `status` reflect the running system, or only our manifest? ·
The log line that proves our thing actually registered: ·

## 16 · Runtime & version lines
What runtime does the platform's *current* release require, and what do users on older runtimes silently receive (`npm view <pkg> dist-tags` / `engines` — pitfall #20)? ·
Which of those lines will we verify against — or all of them? ·

## 17 · Parsers and lifecycle contracts
Which of our declarative files does the host parse strictly, and what does a parse failure look like — error, or **silent absence** (pitfall #18)? ·
Which lifecycle hooks does the host **await** (anything on the startup path must be fire-and-forget — pitfall #19)? ·
Can we register a **mock model provider** (openai-compatible `baseUrl`)? If yes, payload capture makes the whole integration assertable without a live LLM — see the pi crib sheet's harness pattern. ·

---

**Invariant mapping** (from [02 · Portability rules](../docs/02-portability-rules.md) R6): for each plugin invariant, the mechanism here — or the documented degradation:

| Invariant | Mechanism on this platform | Verified? |
|---|---|---|
| · | · | · |

**Target rung** (L0–L4) and what's deliberately deferred: ·
