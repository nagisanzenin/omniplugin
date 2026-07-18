# <Plugin> on <Platform>

<!-- Template for per-platform install docs, distilled from engram's INSTALL-CODEX.md /
     INSTALL-HERMES.md. Keep the section order — especially Honest Status, which is the
     section people actually trust you for. Delete these comments. -->

<Plugin> is an **omni-repo**: one codebase that runs on <list platforms>. The core is the same everywhere — the `skills/` (Agent Skills standard `SKILL.md`) and the dependency-free `scripts/<engine>` are shared verbatim. This file covers the <Platform>-specific glue.

> <One line on how this platform relates to the reference platform, and how many genuine differences there are. If verified live: **say the exact platform version.** e.g. "Everything below was verified against a live <Platform> vX.Y.Z install.">

## What ships for <Platform>

```
<the exact files this platform reads, one line each, SHARED vs platform-specific marked>
```

## Install

### Route A — <the native/plugin route>

```bash
<commands>
```

<What the user sees afterwards: command spellings on THIS platform, e.g. `$learn` not `/learn`.>

### Route B — <the fallback route, always present>

<The skills-only / clone route that works with zero plugin machinery. This is the route that must always work (portability rule R10).>

## The <N> differences on this platform

### ⚠ 1 · <difference>
<What changes, what stays identical, and where the invariant lives now. e.g. "the trigger is manual; the blindness is unchanged.">

### ⚠ 2 · <difference — usually: where state lives / sandbox behavior>
<State location on this platform, env override, prompt/permission behavior.>

## Invoking

| You want | Type | Why |
|---|---|---|
| · | · | <note collisions and escape hatches here> |

## Verify the install

```bash
<the universal engine selftest — same count on every platform>
<a platform-specific dry-run, e.g. piping a fake payload through the hook>
```

## Honest status of the <Platform> glue

**Verified<, live on vX.Y.Z if so>:** <item by item — discovery, command registration, hook firing, wire-level receipts. Only things actually exercised.>

**Not yet verified:** <item by item, each with (a) why the gap is harmless as shipped, and (b) what evidence would close it.> If anything misbehaves, open an issue with what you see.
