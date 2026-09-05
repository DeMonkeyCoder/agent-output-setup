# Rationale: why the setup looks the way it does

`SETUP.md` says what to do. This file says why, for each decision, with the
evidence that drove it. Two nine-stage blind multi-agent reviews produced these
conclusions (2026-09-03 on the Caveman suite, 2026-09-04 on CodeGraph, Context
Mode, and RTK), backed by first-hand probes on a working machine. Where a
conclusion rests on that machine's particulars rather than on the tool, this
file says so.

## The standard everything is judged against

Token efficiency is secondary. The primary criteria are: requirements met,
reasoning that looks ahead, research and result quality preserved, verification
done, and no encouragement to stop at "enough" before the task is satisfied.

This ordering came from an observed regression: an agent stack with several
token-saving tools installed began producing work that looked complete, skipped
verification, and stopped early. The investigation that followed asked, of
every installed tool, not "does it save tokens" but "does it change what the
agent reads, verifies, or decides to stop at". Any tool that does is
*process-changing*, and a process-changing tool needs evidence that requirement
coverage holds with it on before it earns always-on status. Byte savings are
not that evidence.

## Why wording compression is kept, and only at lite

Caveman's `lite` level keeps articles and complete sentences; `full` drops
articles and allows fragments; `ultra` strips conjunctions. Upstream's own text
concedes where that goes wrong: its Auto-Clarity section tells the agent to
abandon compression when "compression itself creates technical ambiguity", and
the maintainer advises "Use lite/off for detail-heavy work". Fragments and
dropped conjunctions are where meaning is lost — a negation, a condition, a
unit. Lite removes filler and nothing else, so it is the only level compatible
with the standard.

Two failure modes shaped the rules beyond the level choice. Negations flip
meaning when dropped, so the rules forbid dropping them explicitly. Article
stripping was measured decaying with session length in one 409-session study
(5.42 articles per 100 words at message 1, 9.15 by message 11+), which is why
the rules say "persist across all turns" and why lite, which keeps articles on
purpose, is less exposed to that drift than full.

## Why only four Caveman skills, and not the fifth

The caveman suite ships around twenty skills. Four are wording helpers
(`-commit`, `-review`, `-help`, `-stats`) and change nothing about process.
The rest were excluded because each one changes process in a way the standard
forbids:

- `caveman-explore` delegates to a small model and demands "ONLY an evidence
  block" as the reply.
- `cavecrew` and its presets delegate to `haiku`; the builder preset has no
  shell.
- `caveman-compress` rewrites memory files; upstream says its own checks "do not
  prove semantic equivalence".
- `lean-build`, `verify-and-stop`, `surgical-patch`, `investigate-first`,
  `safe-refactor`, `migration` each add a stop condition or a scope limit.
  `investigate-first` was the closest call; it embeds a model-judged
  "sufficient evidence" stopping rule, which is exactly the judgment the
  standard does not want automated.

The fifth wording skill, `caveman` itself, was installed after the first review
and removed after the second. Its `SKILL.md` body hardcodes `Default: **full**`,
`ACTIVE EVERY RESPONSE`, and `No preamble, plan, or progress note`. All three
contradict the setup. A skill body loads when the agent judges its description
to match, so the defaults it carries are both wrong and unpredictable. The lite
wording rules belong in the always-on rule file, where they apply every time
and can be read and audited.

## Why the Karpathy guidelines are forked

The upstream file is good and the fork keeps its three core principles nearly
verbatim. The changes are:

**Dropped: "Surgical Changes."** Upstream tells the agent to leave adjacent
problems alone. That pulls in the same direction as the excluded scope-limiting
skills. An agent that notices a related defect should say so, not be instructed
to ignore it.

**Trimmed: "Simplicity First."** Reduced to its lead-in and the senior-engineer
test. The fuller upstream version leans toward doing less, which again overlaps
with scope narrowing.

**Added: precedence over output-style rules.** Caveman says "No preamble, plan,
or progress note before or between calls." Karpathy says "state a brief plan,
each step paired with its verification check." Left unresolved, the agent picks
one silently — and in practice picks the one that produces less text. The fork
says the guidelines win: a brief plan is not narration. This single sentence is
what stops the no-look-ahead failure mode, and it is the most important line
in the setup.

**Added: precedence over tool instructions that narrow evidence.** The second
review found that retrieval tools inject standing text like "don't re-verify
them with grep", "treat each block as a Read you have already performed", and
"return: file path + 1-line description". The original precedence sentence only
covered *style* rules, so these evidence rules had no counterweight and the
model resolved the conflict on its own. The fork now names the pattern —
not-re-verify, not-read, already-read, stop-after-N-calls, path-only reply —
and states that retrieval and compression tools are leads, not verification.
This is written generically so the next tool cannot reopen the gap.

**Added: scope for "ask".** Karpathy says "If something is unclear, stop. Ask."
A separate operating rule for unattended batches says "never stall". Both are
correct in context. The fork resolves them: when the user has explicitly handed
over a batch and left, decide, log each decision with its reason, and report;
everywhere else the ask rule stands.

**Delivery: always-on, never as a skill.** Upstream ships the guidelines as a
skill whose description is coding-scoped ("Use when writing, reviewing, or
refactoring code"). That description does not fire on research, planning, or
diagnosis — which is exactly where the second review found the quality risk
concentrated. Karpathy already prescribes an independent oracle (deterministic
tests) for bug fixes and refactors; the unprotected phases are evidence
gathering, and a coding-scoped skill leaves them unprotected.

## Why CodeGraph is off as an ambient integration

CodeGraph builds a per-repository call graph and exposes one tool that returns
graph-selected source. Its standing instructions, injected into every session
via MCP initialization text and generated rule-file blocks, say:

> Trust codegraph's results — don't re-verify them with grep

> Treat each block as a Read you have already performed: do not Read a file
> shown here

> ONE call usually answers the whole question

Its instruction template states a call budget that nothing enforces, and the
vendor's own design document names the goal as making the agent stop reading.
That is the standard's failure mode, written as an instruction.

Two behaviours were reproduced first-hand on the pinned version and on the next
release. Asked about a symbol name that does not exist, the `callers`,
`callees`, and `impact` paths returned results for the nearest match with no
absence statement — confident wrongness rather than "not found". The default
`explore` path returned related source without stating that the exact symbol
was absent. Exact-name queries, by contrast, were correct with no false
positives, and returned source was byte-identical to disk. So the tool is
accurate when the question is well-formed and misleading when it is not, and
its instructions tell the agent not to check which case it is in.

The evidence for savings is real but regime-bound. The vendor's own A/B matrix
(one run per arm, which it calls "directional") shows fewer reads and calls on
repositories in the hundreds-to-thousands of files, and ties or slower results
on small ones. Output caps are tightest in the under-150-file tier, where
relationship edges and the completeness signal are off by default. The machine
this was measured on had indexes of 2, 2, and 114 files. In that regime the
vendor's evidence shows no gain and the instruction text still applies, so
turning it off costs nothing.

**On a machine with large repositories the calculus is different**, and
`SETUP.md` carves out that case: keep the MCP server so the tool is available
by name, remove the always-on block and hook, and add a rule line that its
output is a lead to be read, not verification. Whether the always-on block
should return on such a machine is an empirical question the reversible
experiment below is designed to answer; the instruction text does not become
acceptable just because the retrieval is faster.

## Why Context Mode is off

Context Mode indexes large tool outputs and returns intent-matched excerpts. Its
routing block, marked MANDATORY and injected by lifecycle hooks into every
session and every subagent prompt, says:

> Do NOT read raw data into context. PROGRAM the analysis, not COMPUTE it.

> ONE call replaces 30+.

> Write artifacts to FILES — never inline. Return: file path + 1-line
> description.

Its benchmark is 21 deterministic fixtures measuring bytes. There is no
end-to-end task study in either direction. Among the three tools it is the one
with the least evidence about task quality.

Two behaviours were reproduced first-hand. A 50 KB output with one decisive line
unrelated to the stated query intent returned a 1.8 KB excerpt without that
line; a later targeted search for the exact string did retrieve it. Omitted
material is retained and recoverable — but only if the agent knows to search
again, and its search throttles after three calls. "Announced and recoverable"
at the tool's layer is silent at the agent's layer, which is what a user
perceives as no look-ahead.

The second finding was discovered because it broke the review's own controls. A
Claude session launched with built-in tools restricted to `Read` still executed
shell commands through Context Mode's `ctx_execute` MCP tool. The `--tools` flag
governs built-in tools only; an MCP server with an execution tool restores a
capability the invocation intended to withhold. That is not a bug in Context
Mode, but it means any reliance on tool allowlists is undone by installing it.

Three open issues confirmed on the pinned version reinforced the decision:
permission anchors `//` and `~/` never match so its deny gate fails open for
them; content databases idle for an hour are deleted on the next session start,
reproduced with data loss; stale cross-session memory can rank above fresh
same-session captures.

## Why RTK is off, and why it is the mildest case

RTK rewrites shell commands through a filtering proxy, transparently: the agent
never sees the rewrite. Unlike the other two, it carries no anti-verification
text. Its filtering is announced in output ("480 matches in 12 files, showing
200") with recovery commands, its contributor policy forbids silent caps, and
first-hand probes on the pinned version confirmed both: grep truncation was
announced with working recovery commands, and `rtk read` was byte-identical to
`cat` on a 1,200-line file. All three of its adapters fail open. It is a
different category from the two above.

It is off anyway, for three reasons.

**The savings claim does not survive its own documentation.** The README says
60–90% token reduction. The docs say bash output is "the only thing RTK
controls", "one contributor to input tokens", and that "a command showing 90%
fewer output bytes does not make your session 90% cheaper". It ships no
tokenizer; `rtk gain` estimates at bytes/4. On the machine measured, its own
estimator reported 5.8% saved over 12,365 commands.

**Independent measurement is neutral to negative.** JetBrains, on Claude Code
with SkillsBench: +7.6% cost at low reasoning effort (p=0.004), ±0% at high
effort, task quality unchanged. A maintainer attributed the cost to extra API
turns when a task is not well covered by the filters. A separate 35-case CI-log
study found downstream diagnosis quality degraded under `rtk-log` with a 13.3%
confident-error rate, recovering only through additional tool calls. Both
studies state their own caveats; neither shows a gain.

**Its transparency creates a gate hazard.** A pattern gate that inspects the
command string sees `rtk git push` rather than `git push`. The Claude hook's
documented response also carries a `permissionDecision` field. Whether approval
systems evaluate the pre-rewrite or post-rewrite command, and whether that
field overrides the user's rules, was not established. Until it is, a
transparent rewrite sits between the agent and every safety gate.

RTK is the likeliest of the three to return, because its hooks are separable
from its text and it fails open. The conditions are in the experiment below.

## Why Ponytail is not installed

Ponytail's skill asks the agent to decide whether a task "needs to exist at all"
and to skip speculative need, choosing "the laziest solution that actually
works". That is a scope-narrowing stop condition of the same family as the
excluded caveman skills, and it was removed on the same grounds during the first
review. It was never wired into a session on this machine after that.

## Why the other installed skills stayed

The second review audited 45 locally installed skills beyond the caveman set
and kept all of them. None is always-on; each loads only when the agent judges
its description to match. Their direction is mostly toward more verification and
look-ahead — "look at X before building around 'X is not possible'", "a
marker-only probe does not validate tools", "implementation-lean output modes
must not govern design work". The few that narrow scope are bound to a named
risky task (device takeover, outbound messaging, destructive database work) and
their gates were kept exactly as written. "Quality" does not authorize bypassing
a safety gate.

## What was learned about the review method itself

Two findings came from the harness rather than the subjects, and both changed
how verification is done in `SETUP.md`.

**A clean exit code is not evidence that the requested model answered.** The
provider silently substituted a different model mid-run after a safety
classifier flagged the packet text. The CLI exited 0 with a normal completion
status and no error; only the native session record showed the change. Every
verification step now reads the native record's model field on every turn.

**Config read-back is not verification.** `--settings` overrides removed hook
*decisions* but not hook *injection*; only an alternative configuration
directory zeroed both. A seat evaluating Context Mode was receiving Context
Mode's own mandatory instructions until that isolation was built. Every
verification step now runs a fresh session and inspects what actually reached
the model.

**One agent's config can be another agent's input.** Cursor loads hooks from
`~/.claude/settings.json` and runs them, merging them below its own. This was
verified, not just read: a `PreToolUse` marker hook placed only in the Claude
file fired during a shell call in the Cursor CLI, and again in a fresh Cursor
IDE install, with no Cursor setting touched in either case and the IDE's own
Hooks tab reporting none configured. The documentation describes this as an
opt-in; on the builds tested it was the default. A hook removed from Cursor's
`hooks.json` but left in Claude Code's settings is still live in Cursor, and
Cursor's UI will not show it. The same shape appeared on the reviewed machine
in a different pair: Claude Code's rule file had leaked into a Hermes profile
through a home-rooted path. This is why `SETUP.md` orders the Claude Code
cleanup before Cursor's and why the verification step reads what reached the
model rather than which file was edited or what a settings screen reports.

## What was and was not tested

Claude Code, Codex, Grok CLI, Hermes, the Cursor CLI, and the Cursor desktop
IDE were tested on a working machine: every removal was followed by a
fresh-session probe and, where the agent keeps one, inspection of the native
session record.

For Cursor, three documentation claims were tested directly. Two held: a plain
`.md` in `.cursor/rules/` is ignored while the same content as an
`alwaysApply: true` `.mdc` loads, and Claude Code hooks fire inside Cursor.
One did not hold as written: the third-party-config opt-in was not required
for Claude hooks on either the CLI or the IDE. The IDE probe also showed the
fork's brief-plan rule in effect unprompted — the agent opened with "Plan: …
Verification: shell output plus tool catalog" before answering.

One thing was not tested: none of the three tools had been installed into
Cursor on this machine, so their removal steps are from each tool's own install
instructions rather than from undoing a real install.

A free Cursor plan cannot pin a model; the answering model is recorded in the
native chat store and was `cursor-grok-4.5-high` for the CLI probes. The IDE
showed `Cursor Grok 4.6 Medium` in its composer for its probe.

## The reversible experiment

A tool returns to always-on status on an agent only after this passes on that
agent. It is read-only: throwaway working and configuration directories, no
persistent change.

**Arms.** Intact current configuration; an isolated configuration with no tool
integrations; and one arm per tool with only that tool added to the isolated
base.

**Tasks.** Three shapes, each with a written requirement list: a coding task
with a breaking change two files from the obvious edit; a research task on a
repository containing one exact symbol and one plausible-but-nonexistent name;
a verification task whose log has one failure, one skip, one strict xfail, one
decisive line past any per-file cap, and a nonzero exit.

**Runs.** Three per task per arm. A run whose model-usage record shows more
than one model is void.

**Scoring.** Requirement coverage against the written list with a four-state
audit so "proven manually" never scores as shipped; a separate confident-wrong
count for any asserted fact the fixture contradicts; deterministic tests for the
coding task; blinded scoring of the research task.

**Pass.** Every stated requirement met, zero confident-wrong assertions, tests
pass, skip and xfail reported accurately, the decisive line present in the reply
or its artifact.

**Interpretation.** If a single-tool arm and the isolated arm pass equally, that
tool's return is evidence-backed on that agent. If the intact arm fails where
the isolated arm passes, the stack is implicated regardless of which single arm
fails.

## What is not claimed

The original regression's cause was never established. Its agent, model, task,
and active integrations were unknown; the review worked from the instruction
text and first-hand probes, and its verdict on that text stands whether or not
any of the three tools was involved in that specific incident. The
recommendation to turn them off rests on what they instruct the agent to do,
not on proof that they did it.
