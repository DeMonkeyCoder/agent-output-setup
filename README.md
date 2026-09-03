# Caveman-lite + Karpathy setup

Token-efficient agent output that does not cost result quality.

Two layers, installed together on one AI coding agent. They are agent-agnostic: the
mechanism is "a rule file the agent always reads", whatever your agent calls that.

| Layer | Source | Job |
|---|---|---|
| Caveman at **lite** | [JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman) | Cut filler from replies |
| Karpathy guidelines | [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills) | Keep the plan and the stated assumptions |

## Why lite, and why only part of the caveman suite

The caveman suite ships ~20 skills. Only the output-wording ones are safe for
quality-sensitive work. Everything else changes *process*: it narrows scope, adds a
stop condition, delegates to a smaller model, or rewrites what the agent reads.

Install: `caveman`, `caveman-commit`, `caveman-review`, `caveman-help`, `caveman-stats`.

Do not install: `lean-build`, `verify-and-stop`, `surgical-patch`, `investigate-first`,
`safe-refactor`, `migration`, `caveman-explore`, `caveman-compress`, `cavecrew` and its
three subagent presets, the six engine driver skills, `native-core.md`, and the
`@caveman-ai/cli` proxy runtime.

Run caveman at `lite`, not the default `full`. Upstream's own intensity table:

- `lite` — "No filler/hedging. Keep articles + full sentences."
- `full` — "Drop articles, fragments OK"
- `ultra` — "Strip conjunctions"

`full` and `ultra` produce fragments and dropped conjunctions, which is where meaning
gets lost. Upstream concedes this: its Auto-Clarity section tells the agent to abandon
caveman when "compression itself creates technical ambiguity", and the maintainer
advises "Use lite/off for detail-heavy work"
([#915](https://github.com/JuliusBrussee/caveman/issues/915)).

## The one conflict you must resolve

Caveman says:

> Tool calls: fire direct. **No preamble, plan, or progress note** before or between calls.

Karpathy says:

> For multi-step tasks, **state a brief plan**, each step paired with its verification check.

Left unresolved, the agent silently picks one. Your rule file must say Karpathy wins:
a brief plan is not narration. This is the single most important line in the setup —
it is what stops the "no look-ahead" failure mode.

Same for hedging: caveman drops it, Karpathy wants "State your assumptions explicitly."
Keep hedges that state real uncertainty; drop only decorative ones.

## Install

### 1. Caveman skills, at lite

Install the five skills through whatever mechanism your agent uses (plugin,
`npx skills add`, or copying `SKILL.md` files into its skills directory).

Set the level to `lite`. Caveman reads, in order: `CAVEMAN_DEFAULT_MODE` env var,
repo-local `.caveman/config.json`, then the user config:

```sh
mkdir -p ~/.config/caveman
printf '{\n  "defaultMode": "lite"\n}\n' > ~/.config/caveman/config.json
```

(`%APPDATA%\caveman\config.json` on Windows; `$XDG_CONFIG_HOME/caveman/config.json`
if that variable is set.)

If your agent has no caveman plugin, skip the skill entirely and put the wording rules
directly in the always-on rule file below. The skill is not the point; the rules are.

### 2. Karpathy guidelines, always on

Copy [`karpathy-guidelines.md`](karpathy-guidelines.md) into your agent's
always-on rule file, or import it from there.

**Always-on is the requirement.** Delivered as a *skill*, the guidelines load only when
the agent judges the skill description to match the task — and the upstream description
is coding-scoped ("Use when writing, reviewing, or refactoring code"), so it will not
fire on research or planning work, which is exactly where you also need it.

Typical carriers:

| Agent | File |
|---|---|
| Claude Code | `~/.claude/CLAUDE.md` (add `@karpathy-guidelines.md`) |
| Codex | `~/.codex/AGENTS.md` (add `@/path/to/karpathy-guidelines.md`) |
| Cursor | `.cursor/rules/karpathy-guidelines.mdc` |
| Grok CLI | `~/.grok/rules/karpathy-guidelines.md` |
| Hermes | a plugin registering a system-prompt section, or `agent.system_prompt` |
| anything else | its global instructions file |

Note the upstream repo ships four principles. This setup uses three: **Surgical
Changes** is dropped because it tells the agent to leave adjacent problems alone, which
pulls in the same direction as the excluded scope-limiting skills. **Simplicity First**
is trimmed to its senior-engineer test for the same reason.

### 3. Remove conflicting instructions

Grep your config for every place that sets a caveman level and change `full` to `lite`.
Check all of them — system prompts, rule files, and **persistent memory files**. A
memory entry saying "caveman full is default" will override a system prompt that says
lite, because the agent reads memory as a user statement of fact.

## Verify

Ask the running agent, in a fresh session:

> Two questions, answer literally and briefly: (1) What caveman intensity level do your
> instructions set, if any? (2) Do your instructions tell you to state a brief plan for
> multi-step tasks? Quote the sentence.

Correct answer: **lite**, and it quotes the brief-plan sentence. Anything else means a
rule file or a memory entry still disagrees. Run this against every agent separately.

## Check it is actually saving you tokens

Caveman adds ~1–1.5k input tokens per turn for the ruleset. If your replies are short,
that costs more than it saves — upstream documents this in
[HONEST-NUMBERS.md](https://github.com/JuliusBrussee/caveman/blob/main/docs/HONEST-NUMBERS.md),
where the output-reduction figure is listed as "Not published".

Run `/caveman-stats`. A negative `Est. net` means turn it off for that workload.

Also know the failure mode: across 409 sessions, one user measured article-stripping
decaying with session length — 5.42 articles/100 words at message 1, 9.15 by message 11+
([#852](https://github.com/JuliusBrussee/caveman/issues/852), fixed upstream in #929).
At `lite` this matters less, since articles are kept deliberately.

## Rollback

One session: say "stop caveman" or `/caveman off`.
Permanently: set `"defaultMode": "off"`, or uninstall the five skills. The Karpathy
guidelines are a plain rule file — delete the import line.

## Provenance

Derived from a nine-stage blind multi-agent review of the full caveman suite at commit
`3b74643f4d910f496babd4e634b1ba7168816f14` (2026-09-02), reading every `SKILL.md`,
the plugin hooks, the native pack, and the upstream benchmarks and issue threads.
