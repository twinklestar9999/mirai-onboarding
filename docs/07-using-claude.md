# Using Claude

How to use an AI coding assistant so it makes you a better developer instead of a
dependent one.

**Time:** ~20 minutes
**You'll need:** Claude Code installed <!-- TODO(team): which AI tools are approved, and
who provides licences? -->

---

## The tension, stated plainly

Claude can write most of a ticket in this program in a few minutes. That is genuinely
useful and it is also the danger, because **the point of the program is not the code. It
is you understanding the code.**

A junior who ships 91 tickets they can't explain has an impressive repo and no ability.
The interview, the on-call page, and the production bug all test the same thing: do you
understand this system? Claude can't attend those for you.

So the rule for this program:

> **Never merge code you couldn't have written, and can't explain line by line.**

Not "never use Claude." Use it constantly. But the code that lands on `main` under your
name is code you understand, because you'll be the one debugging it at 2am.

---

## Where it genuinely helps

Use it freely for these. They cost you time and teach you nothing:

- **Explaining unfamiliar code.** Point it at a file and ask what it does. This is the
  single best use — it's a patient senior developer who will re-explain four times.
- **Understanding an error.** Paste the whole stack trace and ask what it means. Do this
  *after* you've read it yourself, so you're checking your reading rather than skipping it.
- **Boilerplate you already understand.** The tenth CRUD endpoint teaches you nothing the
  first one didn't.
- **Learning a new API.** "Show me how Alembic handles a column rename" beats twenty
  minutes of search.
- **Reviewing your own work before a human does.** "What's wrong with this?" catches the
  embarrassing things.
- **Writing tests for edge cases.** Ask what could break; it's good at the cases you
  didn't think of, which is exactly the list you want.
- **Rubber-ducking.** Explaining your problem out loud works whether or not anything
  answers. Something answering is a bonus.

## Where it will quietly hurt you

Be deliberate about these, especially in epics 02–06:

- **Skipping the struggle a ticket was designed around.** MIRAI-014 asks you to write SQL
  by hand *because* it teaches you what the ORM generates. MIRAI-044 asks you to fetch data
  with `useEffect` *because* the problems you hit are the reason TanStack Query exists.
  Asking Claude to skip to the answer skips the entire ticket. The struggle is the payload.
- **Architecture decisions.** It will give you a confident answer without your constraints,
  your team's conventions, or your deadline. Use it to enumerate options; decide yourself,
  and be able to defend the decision.
- **Anything security-critical.** Epic 04 especially — and MIRAI-034, where a missed
  check means one user reading another's private content.
- **The feed decision in MIRAI-023.** It will confidently pick fan-out on read or write
  without knowing your traffic shape. Ask it to argue both sides; make the call yourself,
  because you're the one defending it. Verify against the actual OWASP
  guidance rather than a plausible-sounding paragraph.
- **Accepting code you don't understand because the tests pass.** Tests passing means it
  does something. It doesn't mean it does the right thing, or that you'll be able to
  change it next month.

---

## How to use it well

### Give it context before asking

The most common bad prompt is a question with no context. Claude Code can read your
files — let it.

Weak:

> how do I add authentication

Strong:

> Read `apps/api/src/routes/posts.py` and `apps/api/src/auth/session.py`. I need to
> require a valid session on the post routes, following the pattern already used in
> `comments.py`. Explain the approach before writing anything.

The second gets you an answer that fits your codebase. The first gets you a tutorial.

### Ask for the explanation first

`/explain` is set up for this. Make "explain the approach before writing code" a habit:
you get to disagree before there's a diff to review, and you learn the reasoning rather
than receiving a result.

### Make it show you the alternative

> What are two other ways to do this, and what's the trade-off?

This is the prompt that turns a code generator into a teacher. It also surfaces when the
first answer was the obvious one rather than the right one.

### Challenge the answer

Claude is confident when it's wrong. Push back:

> Are you sure? What happens if `items` is empty?
> That contradicts what's in `models.py` — check it.

If it folds immediately and reverses itself, that's information about how solid the
original answer was.

### Ask it to find your bug rather than fix it

> Don't fix this. Tell me where the bug is and why, and I'll fix it.

You keep the debugging practice, which is the skill that transfers. Debugging is most of
the job and it is the least delegable part of it.

---

## The rules for this program

1. **Read every line before you accept it.** If you don't understand a line, that's a
   question, not a formality.
2. **You must be able to explain your PR without the transcript.** Your reviewer will ask.
   "Claude wrote it" is not an answer, and it will be obvious.
3. **Never paste secrets** — real credentials, tokens, customer data, production database
   dumps. Same rule as chat. <!-- TODO(team): any restrictions on what code can be shared
   with AI tools? -->
4. **Verify anything factual.** Versions, library APIs, security guidance, cloud pricing.
   Confident and wrong is the failure mode, and it looks exactly like confident and right.
5. **Tests you didn't think about aren't tests.** Generated tests that assert whatever the
   code currently does will pass forever and catch nothing. Read them; ask what each one
   would catch.
6. **When a ticket says "by hand," do it by hand.** Those tickets are marked deliberately.
   You're the only one who can tell whether you skipped one, and you're the only one who
   loses.

---

## In code review

<!-- TODO(team): does the team want AI-assisted work disclosed in PRs? -->

Two things reviewers here care about, whether or not disclosure is required:

- **You understood it.** You'll be asked why something works. Have the answer.
- **It fits the codebase.** Generated code tends to arrive in its own dialect — different
  error handling, different naming, a library nobody else uses. Make it look like the code
  around it before you open the PR.

---

## The honest long view

The developers who'll do best over the next decade are the ones who use these tools
heavily *and* understand their systems deeply. Not one or the other.

The ones who'll struggle are the ones who let the tool do the understanding, because the
understanding is the part that's actually scarce. Code has been cheap for a while now.
Knowing what to build, why it broke, and what it should have been is not.

Use Claude every day. Just make sure that at the end of each ticket, **you** know more
than you did at the start — not just that the repo does.

---

**Next:** [Design with Claude](08-design-with-claude.md) — before you build the UI
