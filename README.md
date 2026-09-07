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

Two columns — **Keep** on the left, **Archive** on the right — and `<` `>` to
stage the end shape. Kept repos hold their ink; staged ones drain to grey, so
the shape of the plan reads without labels. Arrow keys move the selection,
shift and cmd extend it, `/` jumps to the filter.

Committing streams the transfers one repo at a time. Anything that trips a
review rule — five or more stars, a live Pages site, open issues from someone
else, a published package pointing back at it, a protected name — stops and
waits in the **held** pile with its reasons, while the rest keep moving. Each
held repo takes *Archive anyway* or *Keep it*.

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
