# archive

Tooling for keeping [@iamnbutler](https://github.com/iamnbutler) tidy. Dormant
repos get **transferred** to [@iamnbutler-archive](https://github.com/iamnbutler-archive)
and flagged read-only, rather than deleted.

A transfer keeps history, issues, PRs, stars, releases and forks, and leaves a
permanent redirect at the old URL — old links and existing clones keep working.
It is reversible.

**[INDEX.md](INDEX.md)** lists everything archived so far and why.

## Use

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
