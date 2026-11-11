# skills

Agent skills I use every day with Claude Code and Codex, cleaned up for
anyone to use.

Each skill is a folder with a `SKILL.md`, plain Markdown with no scripts or
dependencies. Most of them came out of something that broke, and the Gotchas
sections record what broke and how to avoid it.

> These skills were written with the help of Claude Code. I tested them in my own daily work; the measurements in them
> (timings, failure cases) come from that work.

## Install

With the [`skills`](https://github.com/vercel-labs/skills) CLI (Node.js 22.20+):

```bash
npx skills add MSelmanSonmez/skills
```

Pick one skill:

```bash
npx skills add MSelmanSonmez/skills --skill project-orchestrator
```

Or copy a skill folder by hand:

```bash
git clone https://github.com/MSelmanSonmez/skills.git
cp -R skills/skills/project-orchestrator ~/.claude/skills/
```

For Codex, copy into `.agents/skills/` in your project instead.

## The skills

| Skill | What it does |
|---|---|
| `project-orchestrator` | The agent never edits files itself. It splits work across subagents, asks which model tier to use, verifies results with blinding checks, and keeps a log. |
| `work-cards` | Hands self-contained work to a separate session with a task card, has a fresh agent review it, and asks you before merging. |
| `cli-subagent` | Calls Codex CLI or Antigravity CLI as a one-shot, sandboxed subagent, mostly for image generation or a second opinion. |
| `adopt-external-tool` | Research an outside repo or tool, scope it, dry-run, apply, verify, record. Built for tools that change how your agent works. |
| `brain-file-guard` | Before a structural edit to `CLAUDE.md`, `AGENTS.md`, a skill or a hook: make it reversible, then check the file still works. |
| `runtime-estimate` | Before a long command, state how long it will take and where the estimate comes from; afterwards, record the real time. |

## Gotchas

Each skill has a Gotchas section. A few that cost real time:

- `codex exec` with an open stdin waits for more prompt text and looks hung.
  Give it `</dev/null`. (`cli-subagent`)
- Antigravity CLI (`agy`) ignores "save as X": the image lands under
  `~/.gemini/antigravity-cli/brain/<id>/`. And unless the prompt says to wait for
  the image, print mode exits before it is done. (`cli-subagent`)
- Running a tool's `init --agents claude` and later `init --agents agents` silently
  deleted all the wiring from the first call. Pass every target in one call.
  (`adopt-external-tool`)
- The same shell test suite took about 3.5 minutes on native Linux and between
  35 minutes and 12.5 hours in Git Bash on Windows. Estimate from rows measured on
  the same setup. (`runtime-estimate`)
- `</dev/null` on a heredoc script (`python - <<EOF`) replaces the script with an
  empty stdin, and python sits in its REPL forever. Write the script to a file
  instead. (`project-orchestrator`)

## Requirements

Any agent that reads `SKILL.md` files. `work-cards` works best in the Claude
desktop app (it uses its session tools) and has a fallback for Codex and plain
CLIs. `cli-subagent` needs Codex CLI or Antigravity CLI installed and signed in.

## License

MIT, see [LICENSE](LICENSE). Credits for borrowed ideas are in
[NOTICE.md](NOTICE.md).
