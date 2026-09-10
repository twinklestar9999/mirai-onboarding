# Git workflow

By the end of this page you'll be able to take a change from your machine to an open
pull request without guessing at any step.

**Time:** ~30 minutes
**You'll need:** a working clone from [setup](04-setup.md)

---

## The loop

Every piece of work follows the same six steps:

```
update main  →  branch  →  commit  →  push  →  pull request  →  review  →  merge
```

That's it. It doesn't get more complicated for bigger changes — the changes just get
split into more trips around the loop.

## 1. Start from an up-to-date main

Branching from a stale `main` means resolving conflicts that were never yours.

```bash
git checkout main
git pull
```

<!-- TODO(team): is the default branch `main`, `master`, or `develop`? -->

## 2. Branch

```bash
git checkout -b <type>/MIRAI-<number>-<short-description>
```

<!-- TODO(team): the real branch naming convention, if there is one -->

A reasonable default until told otherwise:

- `feat/MIRAI-031-add-login-endpoint`
- `fix/MIRAI-050-collapse-state-lost-on-rerender`
- `chore/MIRAI-067-pin-base-image-digest`

**Never commit directly to `main`.** If you find yourself on `main` with changes, don't
panic: `git checkout -b my-branch` takes the uncommitted changes with you.

## 3. Commit

Commit small and often. A commit is a save point — one logical change each. "One
logical change" means: if you had to explain it, you'd use one sentence with no "and."

```bash
git status          # what's changed
git diff            # read your own changes before staging them
git add <files>     # stage deliberately — avoid reflexive `git add .`
git commit -m "MIRAI-031: add email format validation to signup form"
```

Message rules:

- Imperative mood, as if completing "This commit will…": *"Add"*, *"Fix"*, *"Remove"*.
- Say **what changed and why**, not how. The diff already shows how.
- Under ~72 characters on the first line. More detail goes in the body after a blank line.

Good: `MIRAI-050: keep collapse state when a thread re-renders`
Bad: `fixed stuff`, `wip`, `changes`, `asdf`

> **Read your own diff before every commit.** `git diff` catches debug prints, stray
> `console.log`s, commented-out experiments, and accidentally staged secrets. This one
> habit removes most first-PR review comments.

## 4. Keep up to date

If your branch lives more than a day, pull `main` into it so conflicts stay small:

```bash
git checkout main && git pull
git checkout <your-branch>
git merge main
```

<!-- TODO(team): does the team prefer merge or rebase for updating a branch? -->

Conflicts are normal and not a sign anything is wrong. Git marks both versions in the
file; you pick what the file should say, remove the `<<<<<<<` markers, then
`git add` and commit. If a conflict is in code you don't understand, ask — resolving
one wrongly silently deletes someone's work.

## 5. Push

```bash
git push -u origin <your-branch>     # first push on a branch
git push                             # every push after
```

`-u` links your local branch to the remote one so later pushes need no arguments.

> **Never `git push --force` to a shared branch.** It rewrites history for everyone and
> can destroy work that isn't yours. If you think you need force, ask first — and if
> you're told to, use `--force-with-lease`, which refuses when someone else has pushed.

## 6. Open a pull request

<!-- TODO(team): PR template? required reviewers? labels? linked ticket? -->

A PR description should answer three questions for the reviewer:

- **What** does this change?
- **Why** — what's the ticket or the problem?
- **How do I check it?** — steps to verify, or a screenshot for anything visual.

Small PRs get reviewed fast and thoroughly. Large ones sit for days and get skimmed.
If yours is over a few hundred lines, ask whether it should be split.

## When it goes wrong

Almost everything in git is recoverable. Before you try anything drastic, ask.

| Situation | Fix |
| --- | --- |
| Uncommitted change you want to discard | `git checkout -- <file>` |
| Committed to the wrong branch | `git log` to copy the hash, switch branch, `git cherry-pick <hash>` |
| Last commit message is wrong (not pushed) | `git commit --amend` |
| Want to undo the last commit, keep the changes | `git reset --soft HEAD~1` |
| Genuinely lost a commit | `git reflog` — it's almost certainly still there. Ask before acting. |

**Never run `git reset --hard` to get out of trouble.** It deletes uncommitted work
with no undo. It's the one command that turns a five-minute problem into a lost day.

---

## Checklist

- [ ] You know the default branch name and the branch naming convention
- [ ] You've made a branch, a commit, and a push
- [ ] You read `git diff` before committing, every time
- [ ] You know what to do — ask — before force-pushing or hard-resetting

---

**Next:** [Code review](10-code-review.md) — what happens to your PR now
