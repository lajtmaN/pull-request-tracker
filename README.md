# prs

A `top`-like terminal board for your open GitHub pull requests. It lists the PRs you authored and the ones
you're asked to review, and tells you for each one **whose move it is** and what the next step is.

```text
YOUR MOVE (2)                WHOSE PR      AGE  STATUS                            TITLE                                               ISSUE
  web-app#412                mine           3h  address changes from alice        feat(auth): remember the last used login method     web-app#398: Faster sign-in
  api#88                     bob            1d  review (asked by bob)             fix(billing): round VAT per line, not per invoice   —

THEIR MOVE (2)               WHOSE PR      AGE  STATUS                            TITLE                                               ISSUE
  web-app#415                mine          40m  alice to review                   feat(settings): add a dark mode toggle              web-app#401: Dark mode
  api#90                     carol          2d  carol to merge (you approved)     chore(deps): bump the HTTP client                   —

RECENT CHANGES
     WHEN  PR                         TYPE         BY             NEXT
  40m ago  web-app#415                requested    you            alice to review
   3h ago  web-app#412                changes req  alice          you to address changes from alice
   1d ago  api#88                     requested    bob            you to review
   2d ago  api#90                     approved     you            carol to merge

synced 12s ago · next sync in 48s (every 60s) · Ctrl-C to quit
```

In a terminal that supports it, every row is a link: click a PR to open it, an issue to open the issue, and a
recent change to jump straight to the review or comment behind it.

## Requirements

- Python 3.9+ (standard library only)
- The [GitHub CLI](https://cli.github.com/), logged in (`gh auth login`)
- macOS or Linux

## Install

```sh
git clone https://github.com/lajtmaN/pull-request-tracker ~/code/prs
cp ~/code/prs/.env.example ~/code/prs/.env    # then edit it
ln -s ~/code/prs/prs ~/.local/bin/prs   # any directory on your PATH
prs test                                 # self-check of the status rules
```

## Usage

| Command | What it does |
|---|---|
| `prs` | Live board: redraws every 5s, syncs with GitHub every 60s. Ctrl-C quits. |
| `prs once` | Prints the board once. Reuses the last sync if it's under 30 minutes old (from any `prs`, including a live view). |
| `prs once --force` | Same, but always syncs first. |
| `prs show` | Prints the board from the saved log, without touching GitHub. |
| `prs sync` | Syncs and prints one line per change (handy in scripts and monitors). |
| `prs log [N]` | The last N changes (default 20), newest first. |
| `prs test` | Runs the built-in checks. |

Piped output (`prs | less`) is always a one-shot board.

## Configuration

Copy [.env.example](.env.example) to `.env` next to the script (it's git-ignored) and adjust it, or set the same
names as environment variables, which take precedence:

```sh
PRS_ORGS=my-org,other-org    # only track PRs in these orgs (default: every repo you can see)
PRS_STRIP_PREFIX=my-org-     # strip a shared repo-name prefix in the PR column
PRS_INTERVAL=60              # live view: seconds between syncs
PRS_MAX_AGE=30               # `prs once`: minutes before a saved sync counts as stale
PRS_ME=my-login              # whose PRs to track (default: the `gh` account)
```

## How it works

Each sync runs three GitHub searches (PRs you authored, PRs where your review is requested, PRs you've
reviewed), compares the results with the previous state and appends what changed to `log.jsonl`. The board is
a replay of that log, so it survives restarts and works offline. The log is private data (PR titles,
colleagues' logins): it's git-ignored, keep it that way.

Whose move it is follows a few rules, in order. For your own PRs: a draft, a merge conflict, failing CI, an
approval, or unanswered feedback means it's your move; pending reviewers mean it's theirs. For PRs you review:
a pending request or new commits/comments since your last review mean it's your move. Bots are ignored. The
full rules are in [CLAUDE.md](CLAUDE.md) and in the `status()` function.

## Claude Code

`CLAUDE.md` tells Claude Code how to read and report the board, and [skills/prs](skills/prs/SKILL.md) is a
`/prs` skill. To use it, link it into your skills folder:

```sh
ln -s ~/code/prs/skills/prs ~/.claude/skills/prs
```

## License

[MIT](LICENSE)
