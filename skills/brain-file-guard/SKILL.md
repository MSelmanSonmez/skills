---
name: brain-file-guard
description: |
  Safety net for structural edits to the files that steer your agent: CLAUDE.md,
  AGENTS.md, skills (SKILL.md) and hooks. Before the edit, make sure the current
  state is committed so it can be restored in one step; after the edit, check the
  file still works (frontmatter parses, hook still runs) and commit the change on
  its own.
  Use before rewriting a large section of CLAUDE.md or AGENTS.md, rebuilding a
  SKILL.md from scratch, or changing a hook's logic: any edit that replaces or
  deletes, not one that only appends.
  Do not use for small appends (adding a line to a notes file or a list). They are
  low risk and this flow would be overhead.
argument-hint: "[file to change] [summary of the change]"
allowed-tools: Bash, Read, Edit, Write
---

# Brain file guard

The flow is three steps: version check, backup, update. In a git repository the
"version" is a commit and the "backup" is a clean, committed state. So the real
work is making sure the file is committed before the change, and proving it is
not broken after.

## Flow

1. **Version check: read the current state.**
   ```bash
   git status --porcelain -- "<file>"
   ```
   - If the file has uncommitted changes (`M` or `??`), they came from an earlier
     session or from you a moment ago. Understand them first
     (`git diff -- "<file>"`). Do not overwrite or lose them. If the user does not
     know about them, ask. If they are yours from this session, continue.
   - If the file is clean, you already have a backup: the last commit is your
     way back.

2. **Backup: commit if needed.** If step 1 found uncommitted work worth keeping,
   commit it in a small separate commit before the structural change. Then the
   structural change sits alone in its own commit, and undoing it (`git revert`)
   or comparing against it (`git diff`) takes one step.
   - Committing needs the user's permission under most setups. Do not commit
     silently; say what you are committing.

3. **Update: make the change.** Apply it with Edit or Write. For a large rewrite
   (rebuilding a whole SKILL.md), read the current file in full first, so you
   know what you are replacing and what will be lost.

4. **Verify: does the file still work?**
   - `CLAUDE.md` / `AGENTS.md`: if a tool manages a block in the file (marked by
     comments such as `<!-- tool:start -->` ... `<!-- tool:end -->`), check that
     you did not touch it. Hand-written sections stay outside those blocks.
   - `SKILL.md`: the frontmatter (`---`, `name`, `description`, `---`) is still
     valid YAML, and `name` matches the directory name.
   - Hooks: the script is still executable (`test -x`) and has no syntax errors
     (`bash -n <script>` or the equivalent for its language).
   - If something broke, the commit from step 2 is your recovery path
     (`git diff`, and if needed `git checkout -- <file>`). Use it.

5. **Commit the change on its own.** When the structural change is done, close it
   with a separate commit (again with permission). The file's new state is the new
   version; the old state stays in history.

## Gotchas

- **Do not edit tool-managed blocks by hand.** If a tool regenerates part of
  `CLAUDE.md` or `AGENTS.md`, a manual edit inside that part can vanish silently
  on the tool's next run. If you need to leave a structural note, add it outside
  the block as a dated section.
- **Do not run this flow for small appends.** Adding a section to a notes file or
  a line to a list does not need it. Use it only for structural, risky changes.
- **No commits without permission.** Steps 2 and 5 need the user's go-ahead; this
  skill never commits automatically.

## Example

To rebuild the "Flow" section of another skill's `SKILL.md`: check with
`git status` that it is clean, read the current file in full, write the new
version, check that the frontmatter still parses and `name` still matches the
folder, then commit with permission.
