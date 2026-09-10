# Epic 02 — Database

**MIRAI-009 – 017** · Postgres, schema design, migrations, and enough SQL to stop
guessing.

Most performance problems and most data bugs are decided here, in the schema, months
before anyone notices them. A social graph punishes bad schema decisions harder than
almost any other domain — slow down on this epic.

---

### MIRAI-009 — Postgres in Docker Compose
**Size:** S · **Depends on:** MIRAI-001

**Goal.** `docker compose up` gives you Postgres and Redis with persistent data.

**Acceptance criteria**
- [ ] Postgres 17 and Redis, with pinned image tags (never `latest`)
- [ ] Named volumes, so data survives a restart
- [ ] Credentials come from `.env`, and `.env` is gitignored; `.env.example` is committed
- [ ] `docker compose down -v` resets cleanly, and you've tested that it does

**You'll learn** — containers, volumes, why pinning image tags avoids a Monday morning
where nothing works and nothing changed.

---

### MIRAI-010 — Connect SQLAlchemy and create the users table
**Size:** M · **Depends on:** MIRAI-009, MIRAI-005

**Goal.** A `users` table defined as a SQLAlchemy model, created by an Alembic migration,
queryable from the API.

**Acceptance criteria**
- [ ] Models in `apps/api/src/db/models.py`, using SQLAlchemy 2.0 declarative style
- [ ] The engine and session are async (`create_async_engine`, `async_sessionmaker`)
- [ ] `id` is a UUID, not a sequential integer — and you can explain why in the PR
- [ ] `handle` is unique, case-insensitive, and validated against a format
- [ ] `email` is unique and case-insensitive
- [ ] `created_at` / `updated_at` with database defaults
- [ ] `GET /users/{handle}` returns a real row

**You'll learn** — connection pooling, UUID vs serial ids (sequential ids tell everyone
how many users you have and let anyone enumerate them), timezone-aware timestamps.

**Hints** — handles need a reserved list (`admin`, `api`, `settings`) or someone will
register `@settings` and break your routing. Decide that now, not after launch.

---

### MIRAI-011 — Migration workflow
**Size:** M · **Depends on:** MIRAI-010

**Goal.** Generate, review, apply, and roll back schema changes as committed files.

**Why.** A schema change that only exists on your machine is a broken deploy waiting to
happen.

**Acceptance criteria**
- [ ] `just db-revision` autogenerates an Alembic migration; `just db-migrate` applies it
- [ ] Migration SQL is committed and reviewed like any code
- [ ] You've written and tested a downgrade
- [ ] Applying migrations twice is safe
- [ ] The README explains the workflow for the next person

**You'll learn** — why you read generated migration SQL before committing it (autogenerate
occasionally proposes dropping a column), forward-only vs reversible migrations.

---

### MIRAI-012 — Design the social graph
**Size:** L · **Depends on:** MIRAI-011

**Goal.** Model users, follows, communities, and memberships.

**Why.** The follow graph is the heart of the product, and it's a self-referencing
many-to-many — the shape most likely to be modelled wrongly by someone doing it for the
first time.

**Acceptance criteria**
- [ ] `follows` links user to user, with the pair unique and self-follows impossible —
      enforced by a CHECK constraint, not by application code
- [ ] `communities` with a unique slug, and `memberships` carrying a role
      (member / moderator / owner)
- [ ] Denormalised counters on users and communities (`follower_count`, `member_count`),
      and the PR explains how they stay correct
- [ ] `blocks` — and the PR states what a block actually does: hides both ways, prevents
      follows, hides replies
- [ ] Every relationship has a foreign key with an explicit deletion rule
- [ ] A diagram in the PR

**You'll learn** — self-referencing relationships, join tables, and the first real
denormalisation decision. Counting followers with `COUNT(*)` works at 100 users and is a
production incident at 100,000; storing a counter is fast and can drift. Pick one, and be
able to defend it.

**Hints** — deleting a user is the interesting case. Cascade, and their replies vanish
from conversations that stop making sense. Consider what "deleted user" should look like.

---

### MIRAI-013 — Design posts, comments, and votes
**Size:** L · **Depends on:** MIRAI-012

**Goal.** The content model — including threaded comments, which is the hard one.

**Why.** `comments.parent_id` referencing `comments` is easy to write and easy to make
unusably slow. How you store a tree determines whether loading a 4,000-reply thread is one
query or four thousand.

**Acceptance criteria**
- [ ] `posts` belonging to a community or to a user's profile, with a body, optional
      media, and a type
- [ ] `comments` self-referencing via `parent_id`
- [ ] A strategy for reading a thread efficiently — adjacency list plus a recursive CTE, a
      materialised path, or `ltree`. **The PR names the choice and its trade-off**
- [ ] Maximum nesting depth enforced, and the PR explains why a limit exists at all
- [ ] `votes` on posts and comments: one vote per user per item, changeable, removable
- [ ] Score stored, not recomputed per read
- [ ] Soft delete for posts and comments — a deleted comment with replies must keep the
      thread intact, showing `[deleted]` rather than orphaning its children

**You'll learn** — trees in relational databases, recursive CTEs, and soft vs hard delete.
The `[deleted]` case is the one people discover in production.

**Hints** — write out, on paper, the exact query that loads a post with its top 50
comments and their replies, *before* you pick a storage strategy. Some strategies make it
one query; others make it impossible.

---

### MIRAI-014 — Write the core queries in raw SQL
**Size:** M · **Depends on:** MIRAI-013

**Goal.** Before touching the ORM's query builder, write these by hand:

- A user's home feed: recent posts from the people and communities they follow
- A post with its top-level comments and their first replies
- A community's posts, ranked by score and recency
- A user's profile: their posts, with vote counts and comment counts

**Why.** Deliberately the harder path. The ORM generates SQL either way — this is where
you learn to read what it generated and recognise when it's doing something absurd.

**Acceptance criteria**
- [ ] Each query written as SQL and run in `psql` first
- [ ] The feed query joins through `follows` correctly, and you can explain INNER vs LEFT
      and which is right here
- [ ] A recursive CTE for the comment tree, if that's the strategy you chose
- [ ] Aggregation with `GROUP BY` for counts
- [ ] **No N+1**: a feed of 50 posts with author, community, and counts is a *bounded*
      number of queries, and you proved it by logging them
- [ ] The PR includes each query with a sentence on what it does

**You'll learn** — JOINs, recursive CTEs, window functions, and the N+1 problem — which is
the single most common performance bug juniors ship, and which a feed makes unmissable.

---

### MIRAI-015 — Port the queries to SQLAlchemy
**Size:** M · **Depends on:** MIRAI-014

**Acceptance criteria**
- [ ] Results are fully typed — `mypy --strict` passes with no `Any` and no `cast()`
- [ ] Relationship loading is explicit (`selectinload` / `joinedload`), never lazy
- [ ] Query logging on in development
- [ ] Generated SQL compared against your handwritten version; any difference explained
- [ ] Still a bounded number of queries per feed page

**You'll learn** — where a query builder helps and where it fights you. Lazy loading is
how an ORM turns your one clean query into fifty without telling you.

---

### MIRAI-016 — Indexes and EXPLAIN at scale
**Size:** M · **Depends on:** MIRAI-015

**Goal.** Seed a realistic social graph, find the slow queries, and fix them.

**Acceptance criteria**
- [ ] Seed scaled to ~10k users, ~500k posts, ~2M comments, ~5M votes, with a *realistic*
      follow distribution — a few accounts with 5,000 followers, most with 20
- [ ] `EXPLAIN ANALYZE` for each core query, before and after, in the PR
- [ ] Indexes on foreign keys, and composite indexes matching your `WHERE` + `ORDER BY`
- [ ] At least one sequential scan turned into an index scan, with timings
- [ ] The feed query for a user following 500 accounts runs in under 100ms
- [ ] The PR explains a column you deliberately did **not** index — indexes cost write
      speed and disk, so indexing everything is its own bug

**You'll learn** — query plans, composite index column order, and why an even distribution
of test data hides every problem real data will cause. The account with 5,000 followers is
the one that breaks your feed.

---

### MIRAI-017 — Transactions and counter integrity
**Size:** M · **Depends on:** MIRAI-015

**Goal.** Multi-step writes either fully happen or don't happen at all — and the
denormalised counters from MIRAI-012 never drift.

**Why.** "Insert a vote, update the post's score" is two writes. If the second fails, the
score is wrong forever, and nobody notices until someone counts.

**Acceptance criteria**
- [ ] Voting, following, and posting each wrap their writes in one transaction
- [ ] A test that forces a mid-transaction failure and asserts nothing was written
- [ ] Concurrent votes on the same post produce the correct final score — tested with
      genuinely parallel requests, not sequential ones
- [ ] A reconciliation job that recomputes counters from source and reports drift
- [ ] The PR explains what a transaction rolls back and what it doesn't (a sent
      notification does not un-send)

**You'll learn** — ACID, row-level locking, race conditions, and that "eventually
consistent counters plus a repair job" is a legitimate engineering answer rather than an
admission of defeat.

---

**Milestone check:** you can explain your schema to someone else in five minutes,
including how you store the comment tree, how counters stay correct, and what happens to a
thread when someone deletes their account.

**Next:** [Epic 03 — API](epic-03-api.md)
