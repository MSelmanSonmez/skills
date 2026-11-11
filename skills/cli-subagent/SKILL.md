---
name: cli-subagent
description: |
  Get work your main agent cannot do (mostly image generation) done by calling
  Codex CLI or Antigravity CLI (`agy`, Gemini) as a subagent: one-shot,
  sandboxed, output in a throwaway folder.
  Use for "have Codex / Gemini generate this image", "ask Codex", "ask Gemini",
  "second opinion", when you need a single image, icon or frame draft, or when a
  hard decision deserves another model's view.
  Do not use for any path that costs money or needs an API key, or when a project
  has a locked visual style that the user did not ask to change.
---

# CLI subagent

Idea from Avenox's video on going from one agent to an agent team ("level 4": hand
a task to another vendor's CLI). Measured on a Windows 11 laptop on
2026-11-06 with `codex-cli 0.166.0`, signed in with a Codex subscription. Check
`codex --version` first; if it is missing, do not install it, ask the user.

## Call

Output always goes to its own folder in your scratch space. It moves into the
project only after the user likes it.

```bash
OUT="<scratch>/codex-<task>"; mkdir -p "$OUT"
codex exec --skip-git-repo-check --ephemeral -s workspace-write -C "$OUT" \
  -o "$OUT/answer.txt" "<task in English>. Save the final image as <name>.png in the current working directory. Reply with the absolute path only." \
  </dev/null > "$OUT/log.txt" 2>&1; echo "exit: $?"
```

- For a question or an opinion only, `-s read-only` is enough and `-C` is not needed.
- Reference image: `-i <file>` (can be given more than once).
- `</dev/null` is required: with stdin open, Codex waits for more prompt text.
- `--ephemeral` keeps Codex from writing a session file under `~/.codex`. The
  generated image still stays under `~/.codex/generated_images/<id>/`; Codex copies
  it into the `-C` folder.
- Forbidden: `--dangerously-bypass-approvals-and-sandbox`,
  `--dangerously-bypass-hook-trust`, and using your repository root as `-C` (Codex
  reads the repo and treats its `AGENTS.md` as its own instructions).

## Measured (2026-11-06, laptop)

| Task | Time | Tokens |
|---|---|---|
| One-sentence question (read-only) | 13 s | 3,308 |
| One image (1254x1254 PNG, ~1 MB) | 56 s | 17,138 |

Tokens come out of the Codex subscription quota, not money. Before a job with
many images, tell the user how many and roughly how long.

## Antigravity CLI (`agy`)

Measured with `agy 1.5.2` on Windows, installed at
`%LOCALAPPDATA%/agy/bin/agy.exe` (not on PATH, so call it by full path), signed in
with a Google account. List models with `agy models`; at the time of measuring
the default was a Gemini model.

```bash
A="$LOCALAPPDATA/agy/bin/agy.exe"; OUT="<scratch>/agy-<task>"; mkdir -p "$OUT"; touch "$OUT/.marker"
cd "$OUT" && "$A" -p "<task in English>. Wait for the image to be ready before ending your turn." \
  --sandbox --mode accept-edits --print-timeout 300s </dev/null > "$OUT/answer.txt" 2>&1
find ~/.gemini/antigravity-cli/brain -newer "$OUT/.marker" -type f \( -iname "*.png" -o -iname "*.jpg" -o -iname "*.webp" \)
```

- The image does not land in the working folder. It goes to
  `~/.gemini/antigravity-cli/brain/<id>/` as a JPG, even when you say "save as X".
  Find the new file with the marker and copy it into `$OUT`.
- Without "Wait ... before ending your turn", print mode exits before the image
  subagent finishes and the answer has no path.
- A question does not need `--mode`. Forbidden: `--dangerously-skip-permissions`,
  `--remote-agent`, running from your repository root.
- Measured (laptop): question 8.5 s; image 39-56 s (1024x1024 JPG). Quota comes
  from the Google subscription, not money.
- Codex or agy: both generate images. If the user says one side's limit is
  running low, use the other. Otherwise prefer Codex, because it puts the file
  where you asked.

## Gotchas

- `codex exec` with an open stdin waits for more prompt text and looks hung; always
  give it `</dev/null`.
- Never point `-C` (Codex) or the working folder (agy) at your repository root: the
  CLI reads the repo and treats its `AGENTS.md` as its own instructions.
- `agy` ignores "save as X" and writes the image under
  `~/.gemini/antigravity-cli/brain/<id>/`; without "wait for the image" in the
  prompt, print mode exits before the image exists.

## Verify

"I saved it" from Codex is not proof. Check that the file exists (`ls`) and look
at its content (open the image). When asking the user to approve an image, show
only the new candidate.

## Privacy

The prompt and any `-i` files go to OpenAI (Codex) or Google (agy). Do not put
personal data, files containing keys, or private notes into the prompt.
