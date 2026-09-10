# Git workflow

Branches, commits, and pull requests — the rules, and the reasoning behind each one.

**Time:** ~40 minutes
**You'll need:** [git setup](09-git-setup.md) done

---

## The loop

Every ticket follows the same six steps:

```
update main → branch → commit → push → pull request → review → merge → deployed
```

It doesn't get more complicated for bigger changes. The changes just get split into more
trips around the loop.

---

## Branches and environments

### The model

`main` is the only long-lived branch. Everything else is a short-lived branch off it.

```
main ─────●────────●─────────●──────────●────────>  always deployable
           \      /           \        /
            ●───●              ●──●──●             feature branches
         MIRAI-023          MIRAI-024              (hours to 2 days)
```

Environments are reached by **promoting the same build**, not by merging between branches:

| Environment | What runs there | How it gets there |
| --- | --- | --- |
| **local** | Your machine, Docker Compose | `just dev` |
| **preview** | One per pull request | Automatic on push (MIRAI-088) |
| **dev** | Shared, latest `main` | Automatic on merge to `main` |
| **staging** | Production-like, real data volume | Promote a dev build — same image |
| **production** | The public site | Promote a staging build, with manual approval |

<!-- TODO(team): confirm which of these environments actually exist, and their URLs -->

**The same container image moves through all of them.** It is built once, on merge, and
tagged with the commit SHA. Nothing is rebuilt per environment — a rebuild is a different
artifact, and then staging didn't test what production runs. That's set up in
[MIRAI-067](epic-07-containers-ci.md) and [MIRAI-078](epic-08-aws.md).

### Why not `develop` and `staging` branches

You'll meet teams with long-lived `develop`, `staging`, and `release/*` branches — the
GitFlow model. It was designed for shipping versioned software on a schedule, and it works
for that.

For a continuously deployed web app it mostly produces problems:

- The same change gets merged three times, and the three copies drift
- "Is this fix in staging yet?" becomes a question with a hard answer
- Long-lived branches accumulate conflicts that nobody owns
- Environments diverge, so staging stops predicting production

If you join a team using GitFlow, use GitFlow — it's their codebase. But understand *why*
they have it, because often the answer is "nobody has changed it since 2016."

### Branch naming

```
<type>/MIRAI-<number>-<short-description>
```

| Type | For |
| --- | --- |
| `feat` | A new capability |
| `fix` | A bug |
| `refactor` | Restructuring with no behaviour change |
| `chore` | Tooling, dependencies, config |
| `docs` | Documentation only |
| `test` | Tests only |

```bash
git checkout -b feat/MIRAI-023-home-feed-ranking
git checkout -b fix/MIRAI-050-collapse-state-lost-on-rerender
git checkout -b chore/MIRAI-067-pin-base-image-digest
```

Rules:

- **Lowercase, hyphens, no spaces or underscores.** Some tooling is case-insensitive and
  some isn't; mixed case eventually causes a confusing failure.
- **Always include the ticket number.** It links the branch to the ticket, the commits, and
  the PR. This is how anyone reconstructs why a line of code exists two years from now.
- **Under about 50 characters.** You'll type it.
- **One ticket per branch.** Two tickets on one branch means neither can merge until both
  are approved.

### Branch lifetime

**Under two days.** A branch alive for a week is a branch collecting conflicts and hiding
work nobody can review.

If a ticket is genuinely bigger than that, split it. A large feature can land across
several merged PRs behind a feature flag, and that's better in every way than one
enormous branch.

Delete the branch after merge — most hosts can do it automatically.

### Never commit directly to `main`

`main` is protected: no direct pushes, PR and passing CI required
([MIRAI-008](epic-01-foundation.md)).

If you find yourself on `main` with uncommitted changes, don't panic:

```bash
git checkout -b feat/MIRAI-023-home-feed-ranking   # takes your changes with you
```

---

## Commits

### Conventional Commits

Every commit message follows this format:

```
<type>(<scope>): <subject>

<body — optional>

<footer — optional>
```

The types are the same as the branch types above. The scope is the area touched: `feed`,
`auth`, `api`, `db`, `ui`.

```
feat(feed): rank posts by score, recency, and author affinity
fix(comments): keep collapse state when a thread re-renders
refactor(api): move vote logic out of the route layer
chore(deps): bump SQLAlchemy to 2.0.36
docs(readme): document the type generation step
test(auth): cover expired session and cross-community access
```

This isn't decoration. It's what generates the changelog and drives version numbers in
[MIRAI-069](epic-07-containers-ci.md).

### Subject line rules

- **Imperative mood** — complete the sentence "This commit will…". `add`, `fix`, `remove`.
  Not `added`, not `fixes`, not `adding`.
- **Lowercase after the colon**, no full stop at the end.
- **Under 72 characters.** Longer gets truncated in most tools.
- **What and why, not how.** The diff already shows how.

| Bad | Why | Better |
| --- | --- | --- |
| `fixed stuff` | Says nothing | `fix(feed): exclude posts from blocked users` |
| `WIP` | Not a commit message | Squash it before opening the PR |
| `updates` | Says nothing | `refactor(db): extract feed query into repository` |
| `fix bug` | Which bug? | `fix(auth): rotate session id on login` |
| `MIRAI-023` | The ticket isn't a description | `feat(feed): add cursor pagination` |

### The body, when you need one

Most commits don't. Use one when the *why* isn't obvious from the diff:

```
fix(feed): use a composite cursor of rank and id

Ranking ties meant two posts with the same score could be returned on
both page one and page two, or skipped entirely. Including the id in
the cursor makes the ordering total.

Fixes MIRAI-024.
```

Blank line after the subject — without it, git treats the whole thing as one long subject.

### Breaking changes

A `!` after the type, and a footer:

```
feat(api)!: return cursor pagination from /feed

BREAKING CHANGE: /feed no longer accepts `page`. Clients must use
`cursor` from the previous response.
```

### Commit hygiene

- **Commit small and often.** One logical change each — if explaining it needs an "and",
  it's two commits.
- **Read your own diff first.** `git diff` before every `git add`. This catches debug
  prints, commented-out experiments, and accidentally staged secrets. It removes most
  first-PR review comments on its own.
- **Stage deliberately.** `git add <files>`, not reflexive `git add .`.
- **Commit at working points**, so you always have somewhere to get back to.

Amend the last commit only if you haven't pushed it:

```bash
git commit --amend
```

Once pushed and someone may have pulled it, amending rewrites shared history. Add a new
commit instead.

---

## Keeping up to date

If your branch lives more than a day, bring `main` into it so conflicts stay small:

```bash
git checkout main && git pull
git checkout feat/MIRAI-023-home-feed-ranking
git merge main
```

<!-- TODO(team): does this team prefer merge or rebase for updating a branch? -->

**Merge vs rebase, briefly:** merge preserves what actually happened and adds a merge
commit. Rebase replays your commits on top of the new `main` for a linear history, and
rewrites your commit ids. Rebase your own unpushed work freely; be careful rebasing
anything someone else may have pulled.

**Conflicts are normal** and not a sign anything is wrong. Git marks both versions; you
decide what the file should say, remove the `<<<<<<<` markers, then `git add` and continue.
If a conflict is in code you don't understand, ask — resolving one wrongly silently deletes
someone's work, and it won't show up as an error.

---

## Pull requests

### PR title

Same format as the commit subject, with the ticket number:

```
feat(feed): rank posts by score, recency, and author affinity [MIRAI-023]
```

<!-- TODO(team): does the ticket number go in the title, or is a linked ticket enough? -->

It becomes the squash commit message on `main`, so a lazy title is permanent.

### PR description

Answer three questions. A reviewer who has to guess any of them reviews worse and slower.

```markdown
## What
One or two sentences on what changed.

## Why
The ticket, the bug, or the decision behind it. Link the ticket.

## How to verify
1. Steps a reviewer can actually follow
2. Or the preview URL, or a screenshot for anything visual

## Notes
Anything a reviewer should know: a decision you're unsure about, a
trade-off you made, something deliberately left for a follow-up.
```

For anything visual, **include a screenshot or the preview URL**. Reviewers approve what
they can see and skim what they have to imagine.

### PR size

**Aim under 400 lines changed.** Small PRs get reviewed within hours and thoroughly. Large
ones sit for days and get skimmed — and a skimmed approval is worse than no review,
because it looks like one.

If yours is bigger, ask whether it should be split. Common splits: schema migration
separate from the code using it; refactor separate from behaviour change; a big feature
behind a flag.

### Draft PRs

Open a draft when you want feedback before it's finished, and say what you want looked at:
*"Draft — is this the right layer for the ranking logic?"* Asking early is cheap. Finding
out after two days isn't.

### Merging

<!-- TODO(team): squash, merge commit, or rebase? Who clicks merge — author or reviewer? -->

Whatever the policy: CI must be green, review approved, and the branch up to date with
`main`. Then verify it deployed and works — see [code review](11-code-review.md).

---

## When it goes wrong

Almost everything in git is recoverable. Before trying anything drastic, ask.

| Situation | Fix |
| --- | --- |
| Discard an uncommitted change | `git restore <file>` |
| Unstage without losing work | `git restore --staged <file>` |
| Committed to the wrong branch | `git log` for the hash, switch branch, `git cherry-pick <hash>` |
| Last commit message wrong (not pushed) | `git commit --amend` |
| Undo the last commit, keep changes | `git reset --soft HEAD~1` |
| Undo a commit already pushed | `git revert <hash>` — a new commit that undoes it |
| Need to switch branches mid-task | `git stash`, then `git stash pop` |
| Genuinely lost a commit | `git reflog` — it's almost certainly there. Ask before acting |

Two rules with no exceptions:

- **Never `git push --force` to a shared branch.** It rewrites history for everyone and can
  destroy work that isn't yours. If you think you need it, ask first — and if told to,
  use `--force-with-lease`, which refuses when someone else has pushed.
- **Never `git reset --hard` to escape trouble.** It deletes uncommitted work with no undo.
  It's the one command that turns a five-minute problem into a lost day.

---

## Checklist

- [ ] You know the branch naming format and always include the ticket number
- [ ] Your commits follow Conventional Commits
- [ ] You read `git diff` before committing, every time
- [ ] Your PRs answer what / why / how to verify
- [ ] You know which environment `main` deploys to, and how staging and production
      get their builds
- [ ] You know what to do — ask — before force-pushing or hard-resetting

---

**Next:** [Code review](11-code-review.md) — what happens to your PR now
