# Code review

By the end of this page you'll know what to do when your PR gets twenty comments, and
how to review someone else's code without being useless or unkind.

**Time:** ~20 minutes
**You'll need:** an open pull request, or someone else's to read

---

## What review is actually for

Review is not a test you pass. It's how the team catches bugs early, spreads knowledge
about the codebase, and keeps the code readable by people who aren't you.

**Your first PR will get a lot of comments.** So will your tenth. That is the system
working correctly. Comments on code are not comments on you — every developer here has
had a PR sent back, most of them this month.

## Before you request review

Review your own PR first, in the web diff. You'll see it differently than in your
editor, and you'll catch:

- [ ] Debug output, `console.log`, commented-out code
- [ ] Secrets, keys, tokens, or a `.env` file
- [ ] Unrelated changes that belong in a different PR
- [ ] Files you didn't mean to touch (formatter noise, lockfiles)
- [ ] Tests — added or updated <!-- TODO(team): what's expected for test coverage? -->
- [ ] CI is green <!-- TODO(team): what runs on CI, and where to see it -->

Then write the description: what, why, how to verify. Then request review.

<!-- TODO(team): how are reviewers chosen — auto-assigned, CODEOWNERS, ask in channel? -->

## While you wait

<!-- TODO(team): expected review turnaround -->

Don't sit idle. Start the next task. If it depends on this PR, branch off your own
branch and mention it — that's normal, just say so in the description.

If a PR has sat with no review past the expected turnaround, **give it one polite
nudge in the channel**. Not a DM to one person — a channel, so whoever is free can
take it. Nudging is expected, not rude. A forgotten PR is much worse than a reminder.

## Responding to comments

**Reply to every comment.** Even just 👍 or "done in `a1b2c3d`." Silence leaves the
reviewer guessing whether you saw it.

Three kinds of comment, three responses:

1. **"Change this"** — make the change, push, reply that it's done.
2. **"Why did you do it this way?"** — usually a genuine question. Answer it. If your
   reason is good, they'll often agree. If while answering you realise there's no
   reason, that's your answer too.
3. **"Consider…" / "nit:"** — a suggestion, not a requirement. Take it or explain why
   not. "nit" means the reviewer thinks it's minor; it's fine to say "leaving as is."

**You're allowed to disagree.** "I tried that, but it breaks X — here's what I mean"
is a completely normal reply. Reviewers are frequently missing context you have. What
you shouldn't do is silently ignore a comment, or silently change something you think
is wrong. If you can't reach agreement in two rounds, move it to a call — text is bad
at disagreement.

**Push fixes as new commits**, not by amending and force-pushing, so the reviewer can
see what changed since they looked. <!-- TODO(team): squash on merge, or keep history? -->

## Reviewing someone else's code

You'll be asked to review sooner than you expect. You are useful immediately — a fresh
reader is exactly who notices what's confusing.

**Look for, in this order:**

1. **Does it do what the ticket says?** Read the ticket first.
2. **Is it correct?** Edge cases: empty, null, zero, very large, concurrent, failed
   network call. This is where real bugs live.
3. **Is it safe?** Any user input reaching a query, a shell, or the page. Anything
   touching auth, permissions, or secrets.
4. **Will I understand this in six months?** Naming, structure, the missing comment
   explaining a non-obvious decision.
5. **Style.** Last, and least — if a linter could catch it, don't spend a comment on it.

**How to write a comment:**

- Ask rather than assert: "What happens if `items` is empty here?" beats "this is
  broken." You may be the one missing something.
- Say which comments are blocking. Prefix optional ones with `nit:`.
- Explain the why, and link a reference if there is one.
- **Say what's good too.** "Nice — I didn't know about this API." It's not padding;
  it tells the author what to keep doing.
- Review the code, never the person. "This function does two things" — not "you always
  do this."

**Saying "I don't understand this" is a valid, valuable review comment.** If you can't
follow it after a real attempt, the next person won't either.

Don't approve something you don't understand. Say what you did check: "I read the
validation logic and it looks right, but I don't know this module well enough to
approve — someone else should look at the caching part."

---

## Checklist

- [ ] You self-review in the web diff before requesting review
- [ ] You reply to every comment, including the ones you disagree with
- [ ] You know how long to wait before nudging, and where to nudge
- [ ] You've reviewed at least one other person's PR

---

**Next:** [Glossary](12-glossary.md) — every term we use that nobody explains
