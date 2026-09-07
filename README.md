# archive

Tooling for keeping [@iamnbutler](https://github.com/iamnbutler) tidy. Dormant
repos get **transferred** to [@iamnbutler-archive](https://github.com/iamnbutler-archive)
and flagged read-only, rather than deleted.

A transfer keeps history, issues, PRs, stars, releases and forks, and leaves a
permanent redirect at the old URL — old links and existing clones keep working.
It is reversible.

**[INDEX.md](INDEX.md)** lists everything archived so far and why.

## The board

```sh
./bin/archive ui        # http://127.0.0.1:8787
```

Three columns — **Keep**, **Archive**, **Delete** — and `<` `>` to stage the end
shape. Kept repos hold their ink, archived ones drain to grey, and the delete
column is hatched, because it is a chute rather than a place things live. Arrow
keys move the selection, shift and cmd extend it, `/` jumps to the filter.

Filters take a small query language, so clearing out forks is one keystroke
rather than twenty-two clicks:

| Query | Matches |
| --- | --- |
| `is:fork` `is:source` | forks / your own repos |
| `is:stale` | no push in `STALE_DAYS` |
| `is:starred` `stars:>5` | by star count |
| `is:public` `is:private` | by visibility |
| `lang:rust` | by primary language |

Anything else is a substring match on name and description, and terms combine.
With a filter active, **select N** takes every match at once.

Committing streams the plan one repo at a time, archives first — those are
reversible, so they should not queue behind a delete waiting on a decision. Anything that trips a
review rule — five or more stars, a live Pages site, open issues from someone
else, a published package pointing back at it, a protected name — stops and
waits in the **held** pile with its reasons, while the rest keep moving. Each
held repo takes *Archive anyway* or *Keep it*; a held delete also offers
*Archive instead*.

## Deleting

Deletion is the one action with no undo and no redirect, so it is gated
differently. It needs a scope the other commands do not:

```sh
gh auth refresh -h github.com -s delete_repo
```

The board disables the delete lane without it. Committing a plan that contains
deletes requires typing the exact number first, and every delete is checked
before it runs — a repo is held if it is not a fork, if the fork carries commits
upstream does not have, if its upstream is gone, if it has forks or a Pages site
of its own, or if anyone else has open issues on it. Stars hold a delete at 1,
except on a clean fork where the bar is 5, since starring a copy of someone
else's project is a bookmark rather than a dependency.

There is deliberately no `delete` subcommand. Destroying a repo should take a
human at the board, not a command an agent can reach for.

The board binds to loopback only and shells out to `gh`. It stages into
`plan.json`; nothing moves until you commit.

## Command line

```sh
./bin/archive check                        # auth, org access, config
./bin/archive candidates                   # repos untouched for a year
./bin/archive candidates --stale-days 180  # cast a wider net
./bin/archive move --reason "2015 coursework" dchack superkids
./bin/archive undo dchack                  # bring one back
./bin/archive index                        # regenerate INDEX.md
```

`move` confirms before it touches anything, and `--dry-run` shows the plan
without making a change. After a successful move it repoints any local clone
under `CODE_DIR` and regenerates `INDEX.md` — commit the result.

## Checks

```sh
./bin/selfcheck
```

Structural only — syntax across all three languages, every `self._method` and
every element id the board reaches for actually resolving, config completeness.
No network. Run it after editing the tool; a missing method otherwise surfaces
partway through a live run.

## Setup

Requires [`gh`](https://cli.github.com) (authenticated) and `jq`.

```sh
brew install gh jq && gh auth login
```

Transferring into an org needs repo-creation rights there. If a transfer 403s:

```sh
gh auth refresh -h github.com -s admin:org
```

Config lives in [`.archiverc`](.archiverc); every value can be overridden by an
env var of the same name.

## For agents

[AGENTS.md](AGENTS.md) has the triage rules — what is safe to archive, what
needs a human yes, and what must never be touched. Read it before proposing a
batch.
