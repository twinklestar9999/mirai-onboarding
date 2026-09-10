# Asking for help

By the end of this page you'll know exactly when to ask, and how to ask so you get a
useful answer on the first reply.

**Time:** ~10 minutes to read. Come back to it every time you're stuck.

---

## Read this part twice

**Asking for help is the job, not a failure at the job.** You were hired knowing you'd
need help. The only way to get this wrong is to stay stuck silently.

Every senior developer on this team gets stuck weekly. The difference isn't that they
get stuck less — it's that they get *unstuck* faster, mostly by asking sooner.

## The 30-minute rule

**Stuck for 30 minutes with no new information? Ask.**

Not "no progress" — *no new information*. If each attempt teaches you something, keep
going. If you're trying variations of the same thing hoping one works, that's the
signal. You've stopped debugging and started guessing.

Three exceptions where you ask **immediately**, no waiting:

- You might have broken something shared — production, staging, a deploy, a shared database
- You might have leaked a secret
- You're blocked on access or permissions you can't grant yourself

## First, spend the 30 minutes well

Do these in order — half the time one of them ends it:

1. **Read the error.** All of it, including the stack trace. The answer is genuinely in
   there more often than feels reasonable.
2. **Search the codebase** for the error text or the function name. Someone has probably
   handled this before.
3. **Search this guide** and the team docs.
4. **Check what changed.** `git diff` and `git log` — if it worked an hour ago,
   something you did is responsible, and it's in that diff.
5. **Reduce it.** Make the smallest thing that still fails. Half the time the reduced
   version makes the cause obvious; the other half, it makes your question much easier
   to answer.
6. **Explain it out loud** to nobody in particular. This works embarrassingly often.

## How to ask

Post in a public channel <!-- TODO(team): which channel for dev questions --> using
this shape:

> **Goal:** what I'm trying to do
> **Tried:** what I've already attempted, and what happened
> **Expected:** what I thought would happen
> **Got:** the actual error, in full, in a code block
> **Ask:** the specific question, or "any pointers?"

Real example:

> **Goal:** run the test suite locally for the first time
> **Tried:** `npm test` after a clean install; also deleted `node_modules` and reinstalled
> **Expected:** tests run
> **Got:**
> ```
> Error: Cannot find module 'pg-native'
>     at Module._resolveFilename (node:internal/modules/cjs/loader:1145:15)
> ```
> **Ask:** is there a system package I'm missing from the setup steps?

That takes four minutes to write and usually gets answered on the first reply. Compare:
"hey, tests aren't working for me, any idea?" — which costs three round trips across
timezones and a full day.

Also: **write it in one message.** Don't send "hi, quick question" and wait. The person
reading it may only be awake for another ten minutes.

## What not to worry about

- **"It's a stupid question."** If you're confused after a genuine attempt, the docs are
  unclear or the code is. Both are worth knowing about. Ask.
- **"I'll look slow."** Asking at 30 minutes looks fast. Surfacing on day three looks
  slow, and by then someone else has to unpick what you built on top of the confusion.
- **"They're busy."** They are. Answering you takes two minutes; rescuing you next week
  takes two days. They'd rather have the two minutes.
- **"I should figure it out myself."** You should try. You did try. That's what the
  30 minutes was.

## After you get an answer

- **Say what worked.** A thread ending in "fixed it, thanks!" with no fix helps nobody
  who searches it later. Post the actual solution.
- **Write it down** if it wasn't documented. A PR against this guide is the ideal
  response to "the setup docs missed a step."
- **Don't apologise for asking.** "Sorry to bother you" trains you to hesitate. "Thanks,
  that did it" is the whole response.

---

## Checklist

- [ ] You know which channel to post development questions in
- [ ] The 30-minute rule is a rule you'll actually follow
- [ ] You've bookmarked the Goal / Tried / Expected / Got / Ask template

---

**Next:** [Using Claude](07-using-claude.md) — the tool you'll use from ticket one
