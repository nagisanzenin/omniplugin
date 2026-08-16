# DeepSeek Harness (dsh) — crib sheet

Dated 2026-08-16, against `@deepseek-ai/dsh` 0.1.0-rc.5 (developer preview — their README
promises breaking changes). Source of truth: engram v1.13.0 port, verified keylessly
against the real npm runtime (skills-in-session + full nudge chain; no model turn ran —
dsh has no free models, needs DEEPSEEK_API_KEY).

## Loader model

Cordis "everything is a plugin"; profiles under `$DSH_HOME/profiles/<name>` compose bundle
patch layers + a user layer (`cordis.patch.yml`). `npx @deepseek-ai/dsh web|--profile
headless "task"`. Headless one-shot exists (`--profile headless`). `--dump-config` prints
the composed tree.

## The four facts that cost the most to learn

1. **Patch grammar: override vs insert.** A bare `- id/name/config` entry is an *override*
   targeting an EXISTING row — naming an unknown id is skipped with only a loader warning
   (looks exactly like success). ADDING a plugin needs `- insert: [ {id, name, config} ]`,
   which fails loud on a missing package. And the profile template ends with a literal
   `[]` that must be REPLACED — appending after it is invalid YAML (loud).
2. **The Claude Code hook bridge discards plain stdout.** `@deepseek-ai/dsh-hooks-claude-code`
   runs unmodified CC hook configs (SessionStart/pre-post-tool/Stop/subagent) but consumes
   ONLY `hookSpecificOutput.additionalContext` JSON. A stock CC SessionStart hook fires
   and delivers nothing — ship a JSON-emitting wrapper (engram: session-start-dsh.sh).
3. **The bridge is NOT in the npm bundle's dependency closure.** `dsh plugin --profile X
   add @deepseek-ai/dsh-hooks-claude-code` AND `... add @deepseek-ai/dsh-hook-protocol`
   (its out-of-closure peer). The plugin command shells out to pnpm (npx shim works:
   `exec npx -y pnpm "$@"`).
4. **SessionStart fires at agent start (first prompt), detached** — before the model call,
   so the whole chain is verifiable WITHOUT an API key: instrument the hook, prompt a
   session, read `$DSH_HOME/sessions/*/*/session.jsonl.zstd` (zstd! python3.14's
   `compression.zstd` decodes) for `agent/inbox/spliced` carrying your context.

## Surfaces

- **Skills**: native SKILL.md directory bundles; frontmatter needs `name`+`description`;
  invocation keys are kebab (`user-invocable`, `disable-model-invocation` — legacy
  camelCase REJECTED). Roots, ranked: project `.dsh/skills`, project `.agents/skills`,
  customSkillDirs, `~/.dsh/skills`, `~/.agents/skills`. Symlinked bundles work. Nested
  `**/SKILL.md` deliberately ignored (an `ln -sfn` into an existing real dir nests one
  level deep and silently disappears — guard install loops). Live-watched (chokidar).
  User gesture: whitespace-bounded `/name` token anywhere in a message; bare names are
  prose. Model gesture: a `skill` loader tool + `<available_skills>` catalog.
- **Instructions**: AGENTS.md + CLAUDE.md, project + user-global `~/.dsh/AGENTS.md`,
  `.local` overlays.
- **Subagents**: TWO default tools and the choice is load-bearing — `subagent` (fresh
  context, "does not see this conversation") vs **`subagent_fork` (seeds the child with
  the whole conversation — silently breaks any blind-grader invariant; forbid it
  explicitly in your skill prose)**. Also claude-code/codex DRIVERS. External plugins'
  agents unregistered by default — construct-isolation-yourself route applies.
- **Version pinning discipline**: the harness and its plugin packages version
  independently (dsh 0.1.0-rc.6 shipped beside bridge 0.0.1-rc.5) — pin and NAME both in
  docs; engram's §7.5 caught the two conflated into one number that was never run.
- **Env**: shell-env plugin manages `DSH_*` keys only (contributor API); no plugin-root
  var for foreign plugins — use a static waterfall path (`~/.agents/<name>` convention).
- **Sandbox**: `DSH_PERMISSION_MODE` default `workspace-write`; writes outside the
  workspace (e.g. `~/.claude/learning`) hit approvals — unverified behavior, flagged.
- **RPC** (web profile): POST `/api/<method>` with
  `{"type":"client-request","rpcId":..,"method":..,"payload":..}` — `session.create`
  {workspace}, `skill.list` {sessionId}, `session.prompt` {sessionId, mode:"queue",
  content:[{type:"text",text}]}. Superb for keyless CI probes.

## Port shape (engram receipt)

Clone to `~/.agents/engram` + symlink skill bundles into `~/.agents/skills` (guarded loop)
+ insert-patch the hook bridge per profile + one waterfall candidate placed LAST (behind
$PWD/git-toplevel — a contributor's checkout must win). Zero adapter code. Receipts:
engram INSTALL-DSH.md, dsh/cordis.patch.yml, hooks/session-start-dsh.sh,
docs/user-sessions/v1.13.0-dsh.md.
