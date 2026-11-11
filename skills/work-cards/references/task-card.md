# Task card

<!-- The brief the dispatcher hands to a separate session. Copy it and fill in
     the brackets. Write it as if the worker cannot ask questions later; close
     ambiguity while writing the card. "Done means" items must be clear enough
     that someone who did not do the work can look and say yes or no ("the report
     is good" does not work; "every claim has a source link next to it" does). -->

**Type:** [code | content]
**Project:** [full path of the folder]
**Waits on:** [name of another card | none]
**Subagent model:** [e.g. a mid-tier model; a small model for mechanical work]. The user's standing choice; do not ask again per subagent.

## Way of working
[Optional. If you use the `project-orchestrator` skill: "Load `project-orchestrator`
first and run the card as an orchestrator: do not touch files yourself, give the
work to subagents, verify the results yourself. Where this card conflicts with
the skill, the card wins."]

## What to do
[One paragraph. Someone reading it can do it without asking.]

## Do not touch
[Out-of-scope files and work.]
- No push, no PR, no merge into the main branch, no file deletion.
- No build in a tree with another session's uncommitted changes (`git status --short` first).

## Done means
- [ ] [What will be true, and what proves it.]
- [ ] [What will be true, and what proves it.]

## If stuck
[Where to stop and report.] When a decision is needed, do not guess; ask with options.

## Records
When the work is done (or stops halfway), write a record, then report. The record
goes to the main checkout, not the worktree:
- One line at the end of your session log:
  `- HH:MM [Card NNN] <what was done>; branch and commit; evidence path; what is left; next step.`
  Take the time from `date +%H:%M`.
- The progress line of the related plan, if there is one.
- Commit only those paths:
  `git -C <main checkout> add -- <paths>` and
  `git -C <main checkout> commit -m "records: Card NNN" -- <paths>`.
  On an `index.lock` error, retry after a few seconds.

## Report
When done, stuck, or when a decision is needed, one sentence: what you did, where
the result is (branch, commit, record commit, the command you ran and where its
output is). Details go in the files, not the sentence.
