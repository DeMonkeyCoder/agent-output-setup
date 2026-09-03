# Cheatsheet

## Install

From [JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman) @ `3b74643f4d910f496babd4e634b1ba7168816f14` (v2.5.0, 2026-09-02):

    caveman  caveman-commit  caveman-review  caveman-help  caveman-stats

## Do not install

    cavecrew            cavecrew-investigator   cavecrew-builder    cavecrew-reviewer
    caveman-compress    caveman-explore         native-core.md      @caveman-ai/cli
    lean-build          verify-and-stop         surgical-patch
    investigate-first   safe-refactor           migration
    caveman-setup       caveman-discover        caveman-learn
    caveman-manage      caveman-optimize        caveman-evidence-review

All of these change process — scope, stop condition, model, or what the agent reads —
rather than wording.

## Configure caveman

`~/.config/caveman/config.json` (`%APPDATA%\caveman\config.json` on Windows):

```json
{
  "defaultMode": "lite"
}
```

Precedence: `CAVEMAN_DEFAULT_MODE` env var > repo `.caveman/config.json` > this file > `full`.
Per session: `/caveman lite`. Off: `/caveman off`.

Never `full` or `ultra` by default — they drop articles and conjunctions.

## Karpathy guidelines

From [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills) @ `2c606141936f1eeef17fa3043a72095b4765b9c2` (2026-04-20):

```sh
git clone https://github.com/multica-ai/andrej-karpathy-skills.git
cd andrej-karpathy-skills
git checkout 2c606141936f1eeef17fa3043a72095b4765b9c2
patch -p1 < /path/to/karpathy-guidelines.patch   # CLAUDE.md -> karpathy-guidelines.md
```

Or just copy [`karpathy-guidelines.md`](karpathy-guidelines.md), which is the result.

Load it from the agent's **always-on** rule file, not as a skill.

## Check

Ask a fresh session: what caveman level is set, and does anything tell you to state a
brief plan for multi-step tasks? Expect `lite` plus the quoted plan sentence.
