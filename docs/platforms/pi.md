# Pi

> Status in engram: **L4 — seventh platform, shipped v1.11.0 + same-day §7.5 patch v1.11.1 (2026-08-04).** Requested by a user ([engram#16](https://github.com/nagisanzenin/engram/issues/16)). The cheapest port to date — no adapter code, no self-extract, no manifest trap: one `package.json` key, one extension file, three prompt templates. Transport verified **20/20 by a committed payload-capture harness** on pi 0.83.0 *and* 0.74.2; a full live tutoring session on pi is the honest gap.
>
> Researched 2026-08-04, against pi 0.83.0 (and 0.74.2, the `legacy-node20` line).

Pi ([pi.dev](https://pi.dev), Earendil) is a deliberately minimal TypeScript coding agent that expects to be extended rather than configured. It reads the Agent Skills standard natively and its package manager understands a plain git repo — the two facts that make it the reference case for how cheap a port *can* be when a platform embraces the standard your core is already written in. No MCP, no subagents, no plan mode, by design; extensions add what's missing.

## Loader model

- Install: `pi install git:github.com/<owner>/<repo>` → clones to `~/.pi/agent/git/<host>/<path>`, runs `npm install` **if a package.json exists** (your devDeps land in the clone — keep the glue zero-dependency), registers in `~/.pi/agent/settings.json`. Also `npm:pkg@1.2.3` (pinned = skipped by `pi update`), bare local paths (referenced, not copied), `-e` for ephemeral try-out, `-l` for project scope.
- Resources declared under a **`pi` key in package.json** — `{"extensions": [...], "skills": [...], "prompts": [...], "themes": [...]}`, globs and `!exclusions` allowed. **No manifest → conventional dirs** (`skills/`, `extensions/`, `prompts/`, `themes/`) are scanned instead; manifest presence disables the convention scan. A repo with no package.json at all still installs by convention — that is how this playbook itself loads on pi.
- Skills: Agent Skills `SKILL.md`, discovered recursively (any dir containing one); listed in the system prompt as agentskills.io `<available_skills>` XML; `/skill:name args` forces invocation; `name:` need not match the directory (pi is deliberately lenient there).
- Prompt templates: `prompts/name.md` → `/name`, with `$@` / `$1` / `${1:-default}` argument substitution. The filename is the command.
- Extensions: TypeScript modules exporting a default factory `(pi: ExtensionAPI) => void`, loaded with **jiti** (ESM semantics — `import.meta.url` works), **re-instantiated per session** (`/new`, `/resume`, `/reload` each build a fresh instance).

## The surfaces that matter

| Need | Mechanism |
|---|---|
| Session-start ambient (the nudge) | `session_start` event (reasons `startup\|reload\|new\|resume\|fork`) — **handlers are awaited before the TUI renders**; fire-and-forget anything slower than microseconds |
| Put text in front of the model | `before_agent_start` → return `{ message }` (persisted, reaches the LLM as a **user-role** message) and/or a chained `systemPrompt` |
| Root env var for shell-outs | set `process.env.YOUR_ROOT` in the factory — both `pi.exec` and the bash tool spawn children from `process.env`, so the export reaches every shell the skills run |
| Headless / spawning children | `pi -p "…"` one-shot; `--mode rpc` (JSONL over stdio, including a **direct `bash` command** that drives the real tool path with no model in the loop); `ctx.hasUI` is `false` in `-p`/`--mode json` |
| User-visible notice | `ctx.ui.notify(text, "info")` — no-op headless, real toast in TUI/RPC |

## No subagents — and it doesn't matter

Pi ships no subagent tool on purpose, but it has the only primitive isolation actually needs: **a fresh process is a fresh context** (R6 — preserve the invariant, not the mechanism). Engram's blind grader runs as:

```bash
ENGRAM_CHILD=1 pi --no-session --no-skills --no-context-files -p "<task>"
```

- `ENGRAM_CHILD=1` + the `ctx.hasUI` check make the plugin's **own ambient extension inert in the child** — a grader's context must not receive the parent's nudge.
- `--no-context-files` keeps the project's AGENTS.md/CLAUDE.md out; `--no-skills` slims the child.
- **Leave extensions on**: custom model providers arrive as extensions, and the child needs whatever provider the parent uses.
- Collect output via a **file the child writes**, never stdout — models garnish stdout.

The shape generalizes: on any platform whose only primitive is a shell, the child is a fresh non-interactive run of *that platform's own* binary.

## The verification pattern worth stealing

Pi lets you register a custom provider in `~/.pi/agent/models.json` (`"api": "openai-completions"`, `baseUrl` at localhost). Point it at a ~100-line mock server that **captures every payload pi was about to send a model**, and the whole integration becomes assertable with no live LLM and no flakiness:

- skills present in the system prompt (and `_shared/` *not* present) — by path, by count, by XML form;
- templates expanding with arguments, in the very first `-p` prompt;
- the injected nudge arriving as a user-role message on a seeded store — and **silence on an empty store asserted over RPC**, the one mode where the machinery actually runs (a print-mode "no nudge" check is vacuous: the extension is inert there for an unrelated reason);
- child hygiene via a **canary**: plant an `AGENTS.md` in the cwd, assert it *present* in the parent-shaped run and *absent* in the child — the flag, not luck, is what excluded it;
- env propagation via RPC's direct `bash` command — the real execution path, zero model.

Engram ships this as [`experiments/pi-harness/`](https://github.com/nagisanzenin/engram/tree/main/experiments/pi-harness) (20 checks, green on 0.83.0 and 0.74.2). The instrument once said no — 13/14 on both pi versions until a frontmatter bug was found — which is what makes its yes worth quoting.

## Gotchas

- **Strict YAML frontmatter, silent skip** (pitfall #18): an unquoted `description:` whose value contains a second colon fails the template's YAML parse, and pi **silently doesn't load it** — `/yourcommand` reaches the model as literal text and nothing anywhere says why. Quote every frontmatter value in `prompts/*.md`. Skills survive more (pi ignores unknown/odd keys there), but the same class applies.
- **`session_start` handlers are awaited** (pitfall #19): pi awaits them before rendering the TUI and before completing `/new` and `/resume`. An awaited `exec` of a slow engine froze startup for the full timeout. Fire the probe, return, let the result land in a `.then`.
- **Killed execs resolve `code: 0`** with `killed: true` — a timed-out child looks like success and can hand you truncated stdout. Check the flag.
- **Engine-gated dist-tags** (pitfall #20): current pi requires Node ≥ 22.19; on Node 20, npm silently serves the `legacy-node20` dist-tag (0.74.2). Say which line you verified — or verify both.
- Print mode: extensions run (UI methods no-op), templates **do** expand in the initial prompt, `ctx.hasUI` is your headless signal.
- The installed npm package ships `docs/` + `examples/` + full `.d.ts` — **version-exact primary sources**, better than the website when behaviors matter (R-rule: read the loader's code).

## Sources

- engram's port: [`pi/engram.ts`](https://github.com/nagisanzenin/engram/blob/main/pi/engram.ts) · [`pi/prompts/`](https://github.com/nagisanzenin/engram/tree/main/pi/prompts) · [INSTALL-PI.md](https://github.com/nagisanzenin/engram/blob/main/INSTALL-PI.md) (honest status itemized) · [`experiments/pi-harness/`](https://github.com/nagisanzenin/engram/tree/main/experiments/pi-harness)
- pi docs: https://pi.dev/docs/latest — extensions, skills, prompt-templates, packages, rpc; repo: https://github.com/earendil-works/pi
