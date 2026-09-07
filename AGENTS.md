# Agent instructions: archiving a repo

You are helping clean up [@iamnbutler](https://github.com/iamnbutler) by moving
dormant repos into the [@iamnbutler-archive](https://github.com/iamnbutler-archive)
org. Everything you need is `bin/archive`; read `bin/archive help` before you start.

## The model

Archiving is a **transfer**, not a deletion. `iamnbutler/foo` becomes
`iamnbutler-archive/foo` and is flagged read-only. History, issues, PRs, stars,
releases, and forks all survive, and GitHub leaves a permanent redirect at the old
URL — existing clones keep fetching, old links keep resolving. It is reversible
with `bin/archive undo <repo>`.

This is why the bar for archiving is low. The only real cost is that the repo
becomes read-only and stops appearing on the main profile. That is the point.

## Workflow

1. **`./bin/archive check`** — confirms auth, org access, and config. If it
   reports the org is missing or your role is wrong, stop and tell Nate; do not
   try to create the org (there is no API for it).
2. **`./bin/archive candidates`** — repos untouched for `STALE_DAYS` (default
   365). Widen with `--stale-days 180` when doing a deeper pass.
3. **Triage the list yourself before proposing anything.** See the rules below.
4. **Propose a batch and wait for Nate's yes.** Group them — "these 12 are
   2013–2016 school projects" reads better than 12 separate asks. Name anything
   you were unsure about and say why.
5. **`./bin/archive move --reason "..." repo1 repo2 ...`** — one `--reason` per
   batch, so give the batch a coherent theme. It prompts before doing anything;
   pass `--dry-run` first if the batch is large or unusual.
6. **Commit** the regenerated `INDEX.md` and `notes.json`:
   `git add INDEX.md notes.json && git commit -m "archive: <theme> (N repos)"`.

## Triage rules

**Archive freely:**
- School and workshop projects, hackathon entries, dated course repos.
- Abandoned experiments and scratch repos with no stars and no README.
- Forks you never pushed to — the upstream is the real thing.
- Superseded work, when a successor repo exists. Say which one in `--reason`.
- Old personal-site iterations that are no longer deployed.

**Ask first, never assume:**
- Anything with **≥5 stars** or meaningful fork counts — someone depends on it,
  and archiving makes their PRs impossible.
- Anything published to a registry (crates.io, npm, PyPI) — check for a
  `Cargo.toml` / `package.json` with a real published name.
- Anything serving a live site (GitHub Pages, a custom domain, `*.github.io`).
  A Pages site on an archived repo keeps serving, but the deploy workflow dies.
- Repos referenced from a résumé, portfolio, blog post, or talk.
- Anything pushed to in the last 6 months, however dead it looks.

**Never archive:**
- `iamnbutler/iamnbutler` (the profile README) or `iamnbutler.github.io`.
- This repo (`archive`).
- Anything with open issues or PRs from other people — resolve or close first;
  archiving freezes them mid-conversation.

## Writing a good `--reason`

It lands in `INDEX.md` and is the only context future-you gets. One clause,
concrete, no filler.

- Good: `2015 YSDN coursework`, `superseded by gpuikit`, `Figma plugin, API long dead`
- Bad: `old`, `cleanup`, `not needed anymore`

## When something goes wrong

| Symptom | Cause | Fix |
| --- | --- | --- |
| `transfer request failed` with a 403 | Token lacks org rights | `gh auth refresh -h github.com -s admin:org` |
| `a repo by that name already exists` | Name collision in the org | Rename the source repo first, or skip it |
| `already flagged archived on GitHub` | Archived in place, not moved | Unarchive on GitHub, then re-run `move` |
| `transfer not visible after 60s` | GitHub is slow, not broken | Wait, then `./bin/archive index` to reconcile |
| Local clone still points at the old URL | No clone under `CODE_DIR` | `git remote set-url origin <new url>` by hand |

Never work around a failed transfer by deleting the source repo. If `move`
cannot finish, report what happened and leave the repo where it is.

## Things not to do

- Do not delete repos. Ever. Archiving exists so deletion is never the answer.
- Do not archive in bulk without a per-batch human yes, even if the list is
  obviously stale.
- Do not hand-edit `INDEX.md` — it is regenerated from the org by
  `./bin/archive index` and your edits will be overwritten.
- Do not run `undo` on your own initiative; it un-archives a repo, which is a
  visible change to a public URL.
