# agent-token-optimization

Token-efficient output from AI coding agents, without paying for it in result
quality. Agent-agnostic; tested on Claude Code, Codex, Grok CLI, Hermes, and the
Cursor CLI.

Two files:

- **[SETUP.md](SETUP.md)** — what to do. Self-contained; an agent can execute it
  alone. Embeds the forked guidelines so nothing else in this repo is needed.
- **[RATIONALE.md](RATIONALE.md)** — why. The evidence behind each decision, what
  was measured on which machine, and what is machine-dependent.

Supporting files:

- `karpathy-guidelines-fork.md` — the always-on guidelines file, as embedded in
  `SETUP.md`. Forked from
  [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills)
  at `2c606141936f1eeef17fa3043a72095b4765b9c2`; the fork drops one principle,
  trims another, and adds three precedence rules. `RATIONALE.md` explains each.
- `karpathy-guidelines-fork.patch` — reproduces the fork from that upstream
  commit: `git checkout 2c60614 && patch -p1 < karpathy-guidelines-fork.patch`
  makes `CLAUDE.md` byte-identical to the fork.

## Tools covered

| Tool | Source | Verdict |
|---|---|---|
| Caveman | [JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman) `3b74643` | Four wording skills at `lite`; the `caveman` skill body and every process-changing skill excluded |
| Karpathy guidelines | [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills) `2c60614` | Forked, always-on, outranks compression and tool text |
| Ponytail | [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) | Not installed: scope-narrowing stop condition |
| CodeGraph | [colbymchenry/codegraph](https://github.com/colbymchenry/codegraph) `v1.5.0` | Off as ambient; binary and indexes kept. Regime-dependent — see `RATIONALE.md` for large repositories |
| Context Mode | [mksglu/context-mode](https://github.com/mksglu/context-mode) `v1.0.169` | Off as ambient; stores kept |
| RTK | [rtk-ai/rtk](https://github.com/rtk-ai/rtk) `v0.45.0` | Off as ambient; binary kept. Mildest case; likeliest to return after the experiment |

Every verdict is against the pinned version named. All six upstreams move; re-read
before adopting a newer one.

## Provenance

Two nine-stage blind multi-agent reviews (2026-09-03 on the Caveman suite,
2026-09-04 on CodeGraph, Context Mode, and RTK), each reading upstream source at
the pinned commit, the installed files on a working machine, vendor benchmarks,
issue threads, and first-hand throwaway-fixture probes of each tool's failure and
recovery behaviour. The findings that came from the review harness breaking —
silent provider model substitution, and an MCP execution tool bypassing a
built-in tool allowlist — are in `RATIONALE.md` under "What was learned about
the review method itself".
