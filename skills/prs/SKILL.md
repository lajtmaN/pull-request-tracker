---
name: prs
description: Show the status of your open PRs (authored + review-requested), split into YOUR MOVE and THEIR MOVE. Use for "/prs", "PR status", "what PRs are waiting on me", "any PR updates", "what's waiting for review".
allowed-tools: Bash(prs *)
argument-hint: '[force]'
---

1. Run `prs once`, or `prs once --force` if the arguments contain `force`/`-f`/`refresh`. It fetches from GitHub only when the last sync by any process (including a `prs` live view in another terminal) is over 30 minutes old; otherwise it reuses that data. `--force` always fetches. It prints the board and exits. Never run bare `prs` here: in a terminal that starts the live view, which never exits.
2. Run `prs log 10`.
3. Reply with exactly:
   - the `once` output **verbatim** in a ```text block (no reformatting, re-sorting or summarising)
   - a second ```text block starting with the line `RECENT CHANGES`, then the `prs log 10` lines **verbatim**, one per line and newest first, the same as the live view. Always keep its first line (the column headers); include only entries newer than the last update you showed in this conversation (all 10 if none has been shown yet). If none are newer, write the plain line `No changes.` instead of the block.

Nothing else. Rules and log format: `CLAUDE.md` next to the `prs` script.
If `once` fails (gh auth, network), show the error line and fall back to `prs show` (the last known board), labelled `(offline)`.
