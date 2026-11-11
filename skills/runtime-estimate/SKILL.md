---
name: runtime-estimate
description: |
  Before running a command that may take a while (test suite, build, full index,
  download, render), tell the user how long it will take and where the estimate
  comes from; when it finishes, report the real time and add it to the table of
  measured runtimes.
  Use before every test suite, build or render, even one you expect to be quick
  (then say "not measured" and estimate anyway), before any other command that
  may run longer than 30 seconds, when writing a prompt that has another agent run a suite or long job, and when the user asks
  "how long will this take".
  Do not use for commands that finish instantly (git status, reading, a single
  file edit).
---

# Runtime estimate

Why: the same suite can take 35 minutes or 2 hours depending on the machine and
shell. Waiting without knowing wrecks the user's plans for the rest of the day.

## Before running

Write one line in the message before the command:

> **Time:** ~N min (source: table, <date> <machine>/<shell>; range best to worst). Background / foreground.

- Take the estimate **from the table**. If the job is not in the table, say "not
  measured" and give a rough estimate with its reasoning (for example "1 run is
  about 30 s, times 4 is ~2 min"). Do not invent a number; give a range if you
  have nothing.
- Be honest about the range: the same job can take 4x longer in another
  environment (see the suite rows below).
- If it will take more than 2 minutes, run it in the background and give the
  finish time as a clock time ("done around 22:40").
- Compare the estimate with your shell tool's timeout (Claude Code: 2 min by
  default, 10 min at most; some harnesses allow 60 min). If it will exceed it,
  start in the background from the beginning and keep the PID.
- If a long job will cross midnight, say so: tests that use the date can break.

## When it finishes

- Write the real time: "Took: N min (estimated M min)".
- If it is more than 50% off the estimate, say why in one sentence (machine busy,
  different shell, network...).
- Add a new row to the table (append only; do not delete old rows). A new
  measurement of the same job goes under the old one.

## Measured runtimes

Keep your own table here (or in a notes file this skill points to). The rows below
are real measurements from the author's machines, kept as examples of what to
record and of how much environment matters.

| Job | Environment | Time | Date / source |
|---|---|---|---|
| Shell test suite (`sh tests/run.sh`, ~1,430 tests) | Windows 10 desktop, Git Bash | 35 min | 2026-10-11 |
| Same suite, run from another agent harness | Windows 10 desktop, Git Bash | 2 h 14 min | 2026-10-13 |
| Same suite (~1,590 tests) | Windows laptop, Git Bash, Claude Code background task, git worktree | 6 h 22 min | 2026-10-14 |
| Same suite (~1,600 tests), another session busy all day | Windows laptop, Git Bash, background task | 10 h 54 min | 2026-10-15 |
| Same suite (1,550 tests), alone | Windows 10 desktop, Git Bash | 12 h 32 min (estimate 2-8 h, missed) | 2026-10-17 |
| Same suite (~1,490 tests), heavy parallel agent load | Windows 10 desktop, Git Bash | ~25 h | 2026-10-16 |
| Same suite | Linux / WSL | ~32 min | 2026-10-10 |
| Same suite (1,592 tests) | Linux laptop, native shell | 3 min 27 s (estimate 10-20 min) | 2026-10-18 |
| Same suite on CI, three OSes | GitHub Actions | ubuntu 3 min 50 s, macOS 8 min 55 s, Windows 67 min 29 s | 2026-10-25 |
| One isolated test block (revert check) | Git Bash | ~50 s | 2026-10-10 |
| `uv sync` with CUDA torch (~4 GB) | laptop, ~2.6 MB/s | 26 min (estimate 45-60 min) | 2026-10-14 |
| Unreal Engine incremental build (2-4 .cpp files) | laptop | 10-22 s | 2026-10-14 |
| Unreal Engine first Shipping cook (`RunUAT BuildCookRun`, ~8,600 packages) | Windows 11 laptop, RTX 4050 | 2 min (estimate 20-60 min) | 2026-11-01 |
| ffmpeg 60 s 1080p60 x264 medium | laptop | ~42 s | 2026-10-14 |
| First `git push` of a repo, 1,284 commits, ~219 MB pack | laptop, Git Bash, SSH | 74 s (estimate 2-10 min) | 2026-10-17 |
| Open-source leak audit agent, 479 commits + 25 tags | laptop, Claude Code | 6 min 22 s (estimate 5-10 min) | 2026-10-24 |
| Image generation, one edit round (Codex built-in image tool) | laptop | 41-52 s (estimate 1-2 min) | 2026-10-22 |

## Gotchas

- **The same suite can differ by 100x across setups.** One shell-heavy suite took
  3.5 minutes on native Linux, about 32 minutes under WSL, and from 35 minutes to
  about 25 hours in Git Bash on Windows. Suite versions differed between runs and
  the cause of the long Windows runs was never isolated. Estimate only from rows
  measured on the same machine and shell.
- **A busy machine is a suspect.** The longest Windows run (~25 h) happened under
  heavy parallel agent load; a run alone took 12.5 h (different commits, so this is
  a suspicion, not a measurement). Before trusting a row, check what else is
  running and whether the laptop is on battery or a power-saving profile.
- **Short jobs get overestimated, long jobs underestimated.** In the table above,
  installs, cooks and pushes beat the estimate by 2x or more, while the long suite
  blew past it. Give a range, not a point.
