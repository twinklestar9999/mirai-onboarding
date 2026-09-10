# The product — Mirai

What you're building across all eleven epics, and what you'll demo at the end.

**Time:** ~20 minutes. Read it properly — everything else in this program exists to get
you here.

---

## The one-sentence version

**Mirai is a social site built around communities: people join communities, post text and
images, reply in threaded conversations, follow each other, and read a personalised feed.**

Think of the shape of Reddit or Threads. Not a clone of either — a real, working, public
site with your name on it.

---

## Why this and not something smaller

Because at the end of seven months you should be able to send someone a link.

Not a repo. Not a screenshot. A URL they can sign up to, post on, and use — that works on
their phone, loads fast, and doesn't fall over. That is the goal, and every epic is
scoped by whether it moves you toward it.

It also happens to be the best teacher available. A social feed forces problems that
smaller projects let you avoid:

| The feature | The problem it forces you to actually solve |
| --- | --- |
| A personalised feed | Fan-out: do you build each user's feed when they read, or when someone posts? This is *the* social-scale question, and it has no free answer |
| Threaded replies | Recursive data in a relational database. Naive approaches die at depth 5 |
| Follows | Many-to-many at scale, and "who sees this?" on every single query |
| Ranking | A real algorithm you design, measure, and defend |
| Image upload | Object storage, presigned URLs, thumbnail generation, and untrusted files |
| Notifications | Fan-out again, plus idempotency — nobody may be notified twice |
| Infinite scroll | Cursor pagination that stays correct while new posts arrive above you |
| Moderation | Trust and safety: reporting, removal, blocking, rate limits, abuse |
| Live updates | Connection lifecycle, reconnection, merging remote changes into local state |

Nothing on that list is decoration. Each maps to tickets, and each is a question you will
be asked in an interview.

---

## What Mirai does

### Accounts and profiles
Sign up with email or with Google/GitHub. Pick a handle (`@yuki`), add an avatar and a
bio. Your profile shows your posts, your communities, and your follower counts.

### Communities
Anyone can create a community — `c/photography`, `c/tokyo`, `c/rust`. Communities have a
description, an icon, rules, and moderators. They can be public or restricted.

### Posts
Post text, images, or a link, either to a community or to your own profile. Posts support
basic formatting, and `@mentions` that notify the person mentioned.

### Threaded replies
Reply to a post, and reply to replies. Threads nest, collapse, and load more on demand —
because a thread with 4,000 comments must not send 4,000 comments to a phone.

### Follows and the feed
Follow people and communities. Your **home feed** is a ranked mix of everything you
follow. **Explore** shows what's active right now across the whole site, so a brand-new
account with zero follows still sees something worth reading.

### Votes
Upvote or downvote posts and replies. Scores drive ranking, and the ranking function is
yours to design and defend.

### Notifications
Someone replied to you, mentioned you, followed you, or your post got traction. In-app,
live, and marked read properly.

### Search
Search posts, people, and communities. Fast enough to type into.

### Moderation
Report a post. Community moderators can remove content, pin a post, and ban a user. Users
can block each other. Rate limits stop the obvious abuse.

---

## The screens

| Screen | What's on it |
| --- | --- |
| **Home feed** | Ranked posts from what you follow, infinite scroll, composer at the top |
| **Explore** | Trending posts and communities — what a logged-out visitor sees first |
| **Post detail** | The post plus its threaded replies, collapsible, load-more at depth |
| **Community** | Header, description, rules, member count, join button, its posts |
| **Profile** | Avatar, bio, follower/following counts, tabs for posts and replies |
| **Notifications** | Grouped, unread-first, live |
| **Search** | Tabs for posts, people, communities |
| **Settings** | Profile, password, linked accounts, blocked users |
| **Auth** | Signup, login, social login, forgot password |

Nine screens. That's the whole site, and it's what [epic 05](epic-05-frontend.md) builds.

---

## Data model

Roughly this. You design it yourself in [epic 02](epic-02-database.md), and it should
change as you learn:

```
users ──< follows >── users            (self-referencing many-to-many)
  │
  ├──< memberships >── communities
  │                        │
  ├──< posts ─────────────-┘
  │      │
  │      ├──< comments (self-referencing: parent_id)
  │      └──< votes >── users
  │
  ├──< notifications
  ├──< blocks >── users
  └──< reports

feed_entries    (per user — if you fan out on write)
sessions        (server-side)
media           (uploads, with their processing state)
```

`>──<` is a join table. The two hard parts are `comments` referencing itself, and
`feed_entries` — whether that table should exist at all is a real decision you'll make in
[MIRAI-023](epic-03-api.md), with consequences either way.

---

## Scope: what's in, what's not

**In scope:** everything above. That's the definition of finished.

**Out of scope** — if you're building these, you've drifted:

- Direct messages
- Video upload (images only — transcoding is a program of its own)
- Live streaming, stories, ephemeral posts
- Recommendation ML — your ranking is a formula you can explain, not a model
- Payments, ads, subscriptions
- A mobile app (the site must work well on a phone browser; that's different)
- Federation / ActivityPub

Scope creep is the most common way a project like this stalls at 70% and never ships.
Ideas that aren't in the tickets go in `IDEAS.md`. Shipping the defined thing is the skill
being taught.

---

## What "finished" actually means

At the end of [epic 11](epic-11-production.md) you should be able to do this, in front of
people, without preparing anything:

1. Open the **public URL** on your phone and hand it to someone
2. They sign up, join a community, and post an image — it works, and it's fast
3. Show them the same post appearing live in your own browser
4. Open your **dashboard** and show their request in the traces: how long it took, and
   which database queries ran
5. Explain your **feed ranking** and why you chose fan-out on read or on write
6. Show the **load test**: where the site breaks, and what breaks first
7. Deliberately **break something**, watch the alert fire, and roll it back

That demo is the deliverable. If you can do all seven, you are employable, and you will
have the answers to most of what an interview can ask.

---

## Definition of done, per ticket

A ticket is done when **all** of these are true — not when the code works on your machine:

- [ ] Acceptance criteria in the ticket all pass
- [ ] Tests written and green, at the level the ticket specifies
- [ ] `just check` passes (mypy, ruff, tsc, biome)
- [ ] Self-reviewed in the web diff
- [ ] PR reviewed and approved
- [ ] Merged, deployed, and verified in the deployed environment

The last line matters. "It works locally" is where junior developers stop, and where
senior developers start checking.

---

## How you'll be assessed

Not on speed. On these, in order:

1. **Does it work, including the edge cases?** Empty feed, blocked user, deleted parent
   comment, unauthorized, concurrent.
2. **Would someone else understand it in six months?**
3. **Did you ask for help at the right time?** Too late is a problem. Too early — before
   reading the error — is also a problem.
4. **Do you apply review feedback, or just this instance of it?** The signal we watch for
   is the same comment not needing to be made twice.

---

**Next:** [Setup](04-setup.md) — get the toolchain running on your machine
