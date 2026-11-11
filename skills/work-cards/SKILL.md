---
name: work-cards
description: |
  Turns the main session into a dispatcher: it does not do self-contained work
  itself (work that will not finish in one turn and has its own branch or output).
  It writes a task card, starts a separate session with it, has a fresh subagent
  review the result, asks the user before merging, updates the records and cleans
  up the leftovers.
  Use as the default whenever the user hands over such work, or says "work card",
  "open a card", "give it to a worker", "do it in a separate session". Also use
  when a note from a worker session arrives.
  Do not use for small work (a few tool calls); do that in the main session. Do
  not use to split one job across subagents inside a single session (that is
  orchestration: see `project-orchestrator`).
---

# Work cards

Idea from Forn's `super-ajan` (MIT): the main session manages, separate sessions
work. Three rules: do not do the work yourself; one card per session, and cards
that do not wait on each other run at the same time; interrupt the user only when
a decision is needed, a card is done, or work is stuck, in one sentence.

## 0. Know your environment

Is `mcp__ccd_session__start_session` in your tool list (or loadable)?
- Yes: you are in the Claude desktop app. Follow steps 1-6.
- No (Codex, Antigravity, plain CLI): steps 1-2 are the same; for starting and
  reporting see "Without the desktop app" at the end. The principles of review,
  merge, records and cleanup stay the same.

## 1. Card, or here?

- Finishes in a few tool calls and has no branch or output of its own: do it
  here, no card.
- Self-contained work: card. If unsure, card.
- If a job needs splitting, the job is one card, not each piece; the worker can
  split it internally.

## 2. Write the card

Fill in [`references/task-card.md`](references/task-card.md): type, project,
waits on, subagent model, way of working, what to do, what not to touch, done
means, if stuck, records, report. Write it as if the worker cannot ask questions
later: close every ambiguity first (if needed, one question to the user, with
options). Each "done means" item must be clear enough that someone who did not do
the work can answer yes or no.

If you copy a brief from another skill, that skill may be stale: check file names,
markers and paths on disk. (Measured case: a copied brief referred to a marker
that no longer existed.)

Save a long card (one with a start condition) as a file in your project's plans
folder first.

## 3. Start the session

`mcp__ccd_session__start_session` (if it is not loaded, load it with `ToolSearch`
and a `select:` query):
- `initiation: "user_asked"` (the user gave the work), `context: "fresh"`,
  `background` of 80+ characters (goal, decisions, constraints).
- Code or config work in this repo: `use_worktree: true` with a short
  `branch_name`. Another repo: `cwd` set to the card's project; for read-only work
  `use_worktree: false`.
- `model`: a mid-tier model for mechanical, well-specified work, effort medium to
  high; the dispatcher stays on the strong model.
- `prompt`: the full card, or the absolute path to the saved card file. If your
  cards use a way-of-working skill (for example `project-orchestrator`), make
  loading it the prompt's first sentence.
- `wake_when_done: true`.
- Right before the call, a bold title line and one sentence; afterwards, a
  progress line in the plan. Do not wait or poll.

## 4. When a note arrives

0. If the worker is waiting for the user's approval or decision (a plan, an image,
   a choice), do not say "it will come to you". Bring the preview here (send the
   image, give the document path), ask the question with options here, and pass
   the answer to the worker with `SendMessage`. (Measured case: the dispatcher
   said "it will come to you" and the user had to go to the worker session and
   approve things there.)
1. Treat the note as a report, not proof: look at the branch, commits and changed
   files yourself (`git log`, `git diff --stat main...<branch>`).
2. **Review:** a fresh subagent (`Agent`, general-purpose; before starting it,
   tell the user its type and tools in one line). Brief: it did not do the work,
   it changes no files, it writes only to scratch space; give it the card's "done
   means" list item by item; ask it to dig into the worker's "could not measure" or
   "note" lines; output PASS/FAIL with evidence and a last line `RESULT:`.
3. Write every "risk/note" line of the report into the plan as
   "risk: ... -> decision: ...".
4. FAIL: send the finding to the same worker with `SendMessage` (`to`: its session
   id). If it does not pass in 3 rounds, ask the user.
5. A worktree does not carry gitignored files, so checks that depend on them can
   give false results in a worktree. Measure in the real checkout instead.

## 5. After PASS

- Content card: done. Code or config card: **ask the user before merging**; with
  approval, `git merge --no-ff refs/heads/<branch>`; then run the project's own
  checks. Push and PR are separate approvals.
- Check that the worker wrote its interim record to the main checkout (see the
  card's "Records" section); if not, ask for it.
- **The dispatcher updates** the records only it owns (session summary, open
  topics, the plan's progress line) and commits only those paths.

## 6. Cleanup

Only after the worker's record and the updates in step 5 are committed:
1. Archive the session (`mcp__ccd_session_mgmt__archive_session`). On Windows an
   open session locks its folder; archiving also removes the worktree. Archive,
   not delete; it can be reopened.
2. Look inside any leftover empty `.claude/worktrees/<name>` folder and remove it
   with `rmdir`; run `git worktree prune`.
3. Delete the branch with `git branch -d`. An unmerged branch will not delete;
   then ask the user.

Before the first cleanup, ask the user once whether steps 1-3 may run without
asking each time. After a yes, run them without further questions.

## Without the desktop app

`start_session`, linked-session notes and the session tree exist only in the
Claude desktop app. In Codex, Antigravity or a plain CLI: start the worker from
the CLI (`codex exec -C <worktree> -o result.md ...`, `agy -p ...`; call details in
the `cli-subagent` skill). The report arrives as a file when the process ends, and
the worker cannot ask questions while it runs, so the card has to be even more
complete. Not yet tested by the author. The plainest fallback: one window per card
by hand, and the dispatcher reads only the card folders.

## Gotchas

- An "offer" card (new work, the user picked "New session") is still a card: the
  brief that goes with its `proposal_id` uses the same template. It is easy to
  forget; a small PreToolUse hook that rejects `start_session` calls whose brief
  lacks a phrase you require (for example the name of the skill the worker must
  load first) catches it mechanically.
- `effort` cannot exceed the main session's: if the main session runs at
  `medium`, a worker asked for `high` still starts at `medium`. After starting, read
  the real value with `get_session` and tell the user that value.
- Timestamps: take them from `date +%H:%M`; never guess the time in a record.
- A rule line that grants the agent a standing permission is rejected by
  automatic permission checks; get the user's explicit approval.
- `SendMessage` "delivered" does not mean read; see the answer in the next note.
- Never send two workers to the same file, and do not edit a file in the main
  session while a worker is writing it.
