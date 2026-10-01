# PR tracker — instructions for Claude

Tracks the user's open PRs (the `gh` account's login, override `PRS_ME`): authored ones, plus ones where
review was requested from them or they have reviewed. Settings live in the git-ignored `.env` (template: `.env.example`)
(`PRS_ORGS`, `PRS_STRIP_PREFIX`, …; see README). `log.jsonl` and `.env` are private: never commit them.

## Files
- `prs` — the CLI (symlink it onto your PATH). Bare `prs` = live board. `once`, `show` (offline), `sync`, `log [N]`, `test`.
- `log.jsonl` — append-only change log, the **source of truth**. One JSON object per line:
  - `{"ev":"set","id","key","ts","f":{changed fields}}` fields: title url role wait why at issue author ci type by link (type = kind of change, link = the review/comment that caused it)
  - `{"ev":"gone","id","key","ts","f":{"why":"merged|closed|untracked"}}`
  - `{"ev":"sync","ts","n"}` heartbeat
  The board is a replay of this log. Never edit or truncate it by hand unless the user explicitly asks
  (then back it up first and rewrite under the log's flock). To restore after
  a lost session or in the morning, just run `prs`: it replays the log and syncs.

## Board
Sections: YOUR MOVE (n) and THEIR MOVE (n), each with its own column header row; "synced … ago" is the last line (live: in the footer):
PR · WHOSE PR (`mine` or the author) · AGE (since last human activity) · STATUS (≤32 chars, cut with …) ·
TITLE · ISSUE (the PR's Development-section issue, `—` if none). Rows newest activity first.

## Status rules (the `status()` function, run `prs test` after changing it)
STATUS text: under YOUR MOVE an instruction to the user, under THEIR MOVE "<who> to <verb>"; reasons go in
parentheses so `next_step()` can strip them. Role internally: A = the user authored it, R = they review it.
Author (A), first match wins: draft → ME "finish draft" · conflict → ME "resolve conflict" ·
CI red → ME "fix failing CI" · approved (or, without required reviews, an approval and no open
changes-requested) → ME "merge (approved by X)" · last review/comment/push by someone else → ME
"address changes from X" / "reply to X" / "check commits from X" · pending reviewers → THEM "X to review" ·
earlier feedback answered, nobody requested → THEM "X to re-review" · else ME "request review".
Reviewer (R): draft → THEM "bob to finish draft" · requested → ME "review (asked by bob)" ·
author commented/pushed after my review → ME "reply to bob" / "re-review (bob pushed)" ·
else THEM "bob to merge (you approved)" / "to address your review" / "to reply to you" / "to act (not your review)".
Bots are ignored. While GitHub reports mergeability as UNKNOWN, a conflict state is kept (no flip-flop).

## Recent changes line
`WHEN  PR  TYPE  BY  NEXT` under a RECENT CHANGES heading. WHEN = when the GitHub event happened
(sync time for state changes like CI), newest first. TYPE = what happened. BY = who did it (`you`, a
login, `github` for CI/conflicts, `—` unknown; the log stores `me`). NEXT = the step it left the PR on,
without the reason: "bob to review", or "you to …" when it is the user's move. Merged/closed have no NEXT.
Types: approved, changes req, commented, dismissed, pushed, requested, ready, opened, tracked,
CI failed, CI fixed, CI running, CI passed, CI started (hidden), conflict, no conflict, draft, retitled, changed, linked, unlinked,
merged, closed, untracked.
Live view colors: WHEN/AGE green <1d, yellow <1w, red older; TYPE by kind; BY per person; your-move
text bold yellow; each issue its own color.
When the status texts change, bump `WORDING` in `prs`: the next sync logs the rewording as hidden
`reworded` lines. A newly added field is filled the same way with hidden `backfill` lines.

## When the user asks for an update ("prs", "status", "what's waiting")
1. Run `prs once` (bare `prs` is the live, top-like view for terminals). It syncs only if the last sync
   by anyone is over 30 min old (`PRS_MAX_AGE`), so it reuses a running live view's data; `--force` always syncs.
2. Paste its output **verbatim** in a ```text block. Do not reformat it, re-sort it, or summarise it.
3. Then a second ```text block, `RECENT CHANGES`, holding the `prs log 10` lines verbatim (one per line,
   newest first), limited to entries newer than the last update they saw, or `No changes.` Nothing else.
   The `/prs` skill (`skills/prs/SKILL.md`) has the exact wording.

## Monitor
Nothing runs in the background unless a Claude session has armed one. When no monitor is
running, re-arm this Monitor (30 min timeout, re-arm on expiry):
`while true; do prs sync 2>&1 | grep -E --line-buffered ' you to |  merged|  closed|  untracked|Error|Traceback|not found'; sleep 300; done`
Send a PushNotification only when a line says `you to …` (a PR became the user's move).
