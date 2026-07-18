# 05 · Pitfalls — field notes with receipts

Every entry: what actually happened → the rule it taught. All from [engram](https://github.com/nagisanzenin/engram)'s history; dates and versions are real.

---

## 1 · The port built by analogy

**What happened:** an external Antigravity PR (engram #8) shipped a `commands/` directory and a manifest with `skills`/`agents`/`commands`/`version` fields — all reasonable *by analogy to Codex*. On the actual target: AG's schema allows exactly `{name, description}` (`additionalProperties: false`), discovery is by directory convention, commands don't exist as a concept, and skills auto-derive slash commands anyway. Nearly the entire contribution was dead weight on the platform it targeted.

**Rule:** research the target's actual loader from primary sources (schema, CLI changelog, flagship plugin) before writing or accepting a line of glue. A port's correctness is a property of the *target platform*, not of the diff.

## 2 · Namespace bleed: the file that did nothing on its platform and broke another

**What happened:** that same `commands/` directory — inert on Antigravity — would have been auto-discovered by **Claude Code**, which scans root `commands/` with no manifest declaration. Every existing CC user would have received a stray `/engram:review-loop` on their next update, with the `learn`/`coach` stubs silently shadowed by same-named skills (skill takes precedence).

**Rule:** platforms scan overlapping root namespaces. Every new root path gets checked against every *shipped* platform's discovery conventions, not just the target's.

## 3 · Version drift multiplies per platform

**What happened, three ways:** the README version badge was missed in v1.0.3 *and* v1.0.4; a stale `package.json` would make npm publish old code under a new tag; the AG PR's optional `version` field would have created a fourth manifest location that the release-grep didn't scan.

**Rule:** the bump is a grep with an authoritative location list; a selftest pins engine-version == manifest-version; optional version fields are refused. ([04 · Maintenance](04-maintenance.md) §1.)

## 4 · The installer that severs your core

**What happened:** Hermes' hub installer copies each skill folder plus only files referenced *inside it* — "unreferenced repository files are not copied." Engram's skills share a repo-level engine and `skills/_shared/`; hub-installed copies would be skills with no engine, a breakage no error message would ever explain.

**Rule:** the staging model (whole-repo vs referenced-files-only) is intake question #7, answered before the install doc is written. Where staging severs, the install route is clone + external-dirs — and the doc says *why*.

## 5 · The ambient hook that failed open

**What happened:** the Hermes `pre_llm_call` hook — contract: "at most one nudge per session, silence on any failure" — reached review failing open two ways: an empty `session_id` bypassed the dedupe entirely (nudge on every LLM call), and a misplaced `2>/dev/null` made an unwritable TMPDIR both leak bash errors and re-nudge per call.

**Rule:** ambient adapters fail **closed**: can't identify the session → at most once per process; can't write the dedupe marker → silence. "Silence over repetition" is the contract, and it gets a failure battery (9 cases) shipped with the hook.

## 6 · Your command name is someone's built-in

**What happened:** `/learn` is Hermes' flagship built-in (authors new skills). And after the collision was found, documented, and the hook taught to rewrite its nudge to `/skill learn` — a review *still* caught the cron delivery path emitting the un-rewritten `/learn`, two sections after the doc warned about exactly that.

**Rule:** collision handling is: detect → escape hatch → rewrite your own strings in **every** output path → document per-platform spelling in the matrix. ([02 · Portability rules](02-portability-rules.md) R5.)

## 7 · The platform you can't run

**What happened:** the Codex port was built without a live Codex binary. It shipped anyway — legitimately — because every unverified path degrades harmlessly (self-resolving hook, silence on failure), a skills-only Route B carries the whole loop, and the install doc splits Verified / Not-verified item by item and asks users to report.

**Rule:** "can't verify" is shippable; *pretending* you verified is not. Harmless degradation + honest status + a fallback route is the template. ([03 · Adding a platform](03-adding-a-platform.md) Phase 4.)

## 8 · The platform whose cache isn't a config dir

**What happened:** OpenCode loads npm-installed plugins from its npm cache, which its discovery never scans for skills/agents — so a "correct" package was invisible. The fix was a real adapter: self-extract into the config dir on first run, a first-session config bridge so everything works *immediately* (not after a restart), `copyMissing` so user edits are never overwritten, and a deterministic `/engram-update` flow (byte-compare diffs, allowlisted deletes, path-traversal guards, zero bash in templates) because "just re-extract" would clobber users.

**Rule:** when a platform forces real adapter code, the adapter inherits product-grade obligations: its own test suite (88 vitest), CI, an update story that respects user edits, and security posture (no interpolated shell, validated paths).

## 9 · Cross-platform prose drift

**What happened:** one release review found five stale cross-platform claims in the README at once — "four platforms" counting one still in review, a missing second install command in a matrix cell, dropped platform-specific details, an example unscoped under a matrix that had just grown a row, and a nudge-coverage claim that had silently become true of more platforms than it named.

**Rule:** every sentence containing a platform name or count is re-read on every release. The matrix is load-bearing documentation, not decoration.

## 10 · Sandbox and environment asymmetries

**What happened, small but real:** Codex's default `workspace-write` sandbox prompts on state writes outside the workspace (state dir placement must be documented per platform); Hermes' headless `chat -q` passes slash commands through as literal text (so the obvious smoke test is a false negative — the doc says how *not* to test); Hermes state lives on whatever host runs the terminal backend, which changes what "shared schedule" means on remote backends.

**Rule:** intake questions 9 and 11 exist because each of these cost a debugging session. Answer them during research, not from bug reports.

## 11 · Trigger semantics differ even when formats port cleanly

**What happened:** Claude Code auto-delegates subagents ("MUST BE USED" descriptions work); Codex spawns TOML agents **only on explicit request**; Hermes needs `delegate_task` with the prompt passed as context. The same assessor prompt ports everywhere — but a skill that says "spawn the assessor" silently does nothing on platforms without auto-routing unless the skill/docs carry the per-platform invocation.

**Rule:** port the prompt, adapt the trigger, preserve the guarantee — and verify the *invocation path*, not just the file format. ([02 · Portability rules](02-portability-rules.md) R6.)

---

## The meta-lesson

Engram's release protocol states it once and it applies double across platforms: **every gate you build misses its own bug class — the only things that ever found the real bugs were a fuzzer, an outside reviewer, and a real user.** With N platforms you cannot even run everything you ship, so the outside world isn't just helpful, it's structural: design for harmless degradation, publish honest status, and treat every user report from a platform you can't run as the integration test it is.
