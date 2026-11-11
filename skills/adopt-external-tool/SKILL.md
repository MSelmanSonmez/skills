---
name: adopt-external-tool
description: |
  Research an outside project, repo or CLI tool and adopt it into your project
  safely and with the narrowest scope: research, think about scope, dry-run,
  apply, verify, record. The gotchas below (a dry-run that behaves differently from
  the real run, two separate `init` calls undoing each other) were measured while
  installing real agent tools in September and October 2026.
  Use when someone shares a GitHub, npm or CLI link and asks "what does this do",
  "is it useful for us", "look into it and set it up", especially when the tool
  writes to `.claude/`, `AGENTS.md`, `CLAUDE.md`, global config, or registers hooks
  or MCP servers, i.e. anything that changes how your agent works.
  Do not use when the tool and the install command are already settled and only a
  mechanical install is asked for (just do it), or when the content is a concept
  article or video rather than code.
---

# Adopt an external tool

When you bring something in from outside ("what is this project, is it useful,
set it up"), the main risk is scope, more than the install itself. An agent or
CLI tool often picks silently between "this repo only" and "every repo on this
machine", and its documentation does not always match what it actually does.

## Flow

1. **Research.** Read the README and repo metadata (`gh api`, a web fetch): what it
   does, who maintains it (stars, last commit, license), whether the claimed
   benefit is measured or marketing. Does it run locally and for free, or does it
   need a key or network calls?
2. **Think about where it fits.** A repository often mixes notes or personal
   content (which the tool should not touch) with real code (where the tool
   actually helps). Which subset should the tool be scoped to: the whole repo, or
   one subproject?
3. **Ask the scope questions.** These are real choices a tool cannot make for you.
   Ask the user:
   - repo-scoped, or machine-wide (`~/...`, global config)?
   - repo root, or one subproject?
   - show only the `--dry-run` / plan first?
4. **Protect the agent's own files first.** If the tool will write to
   `.claude/settings.json`, `.mcp.json`, `AGENTS.md`, `CLAUDE.md`, a hook or
   `.gitignore`, make sure that file is committed and clean before it runs (the
   `brain-file-guard` skill).
5. **Do not trust the dry-run; read the source.** A tool's `--dry-run` output can
   contradict the real write. Measured case: with `--no-global`, one tool's
   dry-run kept listing global writes, while the real run skipped them correctly.
   Reading the code settled it. When the behavior is unclear or risky, read the
   relevant function in the installed package (`grep`, `sed`). Trust the code over
   the docs.
6. **Start from the narrowest scope.** Prefer repo-scoped over global, sudo or
   machine-wide, and flags like `--no-global` / `--local` when they exist. If a
   sudo password prompt blocks a global install, try an install-free path such as
   `npx`.
7. **Give all targets (agents, integrations) in one call.** Some tools undo the
   wiring of any target you did not select. Measured case: running
   `init --agents claude` and then a separate `init --agents agents` silently
   deleted all of the Claude wiring from the first call. Before adding a second
   integration, check for this remove-and-reinstall behavior, and pass everything
   in one call when you can (`--agents claude agents`).
8. **Apply.**
9. **Verify.** Show with `git diff` that the written config is still valid JSON or
   syntax and that earlier hooks or managed blocks are untouched. Then run a real
   functional smoke test: "the file was written" and "it works" are different
   claims.
10. **Commit with permission, in separate logical groups.** Never put the files
    this work touched in the same commit as unrelated pending changes.
11. **Record what you found.** Tell the user plainly which parts were unsupported
    or skipped (an integration point the tool does not know about, say) and which
    gotchas you hit. If the behavior will come up again, write it down where your
    project keeps lasting notes.

## Gotchas

- **Dry-run is not the real behavior.** A flag may affect only the real write and
  not the dry-run output. Do not mix the two up; read the source if needed.
- **Separate `init` or install calls can undo each other.** A tool may "clean up
  unselected targets", so adding a new integration can silently remove an old
  one. Try giving everything in one call; if that is not possible, notice the
  removal in the same session and fix it right away.
- **"Global" means different things per tool.** Some integration points (an MCP
  registration in a CLI, for example) have no repo-scoped option at all, and
  `--no-global` disables that integration entirely instead of narrowing it. Tell
  the user the integration is inherently global; do not choose silently.
- **The first `build` or `sync` after install can rewrite the wiring** (measured
  on Windows, October 2026). In one tool, the first build reset `.mcp.json`,
  `settings.json`, `AGENTS.md` and helper files to the default folder name and
  reverted a folder choice made earlier. After the build, check `git status` and
  restore unexpected changes with `git restore`.
- **On Windows, an npm-installed CLI used as an MCP command** must be given as
  `cmd /c <tool> ...`. A bare name (or `npx`) cannot start without the `.cmd`
  shim.

## When not to use

- The tool and command are already decided and the install is one mechanical
  step.
- The content is not code or a CLI (an article, a video, a concept). Normal
  research is enough.
- It is a Claude Code plugin (it has `.claude-plugin/plugin.json`). Decide
  separately whether to install it as a plugin or copy only its text parts
  (skills, commands, agents) into your project.
