# Epic 03 — API

**MIRAI-018 – 028** · A REST API you'd be comfortable handing to another team.

By the end of this epic you hit **Milestone M1**: a working social network with no UI,
drivable entirely from `curl`.

---

### MIRAI-018 — Layer the application
**Size:** M · **Depends on:** MIRAI-015

**Goal.** Split the code into routes → services → repositories, with rules about which
layer may import which.

**Why.** Everything after this lands in one of these layers. Deciding the boundaries now
is why the codebase is still navigable at ticket 101.

**Acceptance criteria**
- [ ] Routes handle HTTP only: parse, call a service, format the response
- [ ] Services hold business rules and know nothing about HTTP
- [ ] Repositories hold database access and are the only thing importing SQLAlchemy
- [ ] No FastAPI `Request`/`Response` object, and no `HTTPException`, below the route
      layer — services raise domain errors instead
- [ ] The PR states the import rules in one line each

**You'll learn** — separation of concerns, and why a service that knows about HTTP can't
be reused by a background job, a CLI, or the feed rebuilder you'll write in MIRAI-023.

---

### MIRAI-019 — Request validation
**Size:** M · **Depends on:** MIRAI-018, MIRAI-004

**Goal.** Every endpoint validates body, params, and query through Pydantic models.

**Acceptance criteria**
- [ ] Request models declared as type hints — FastAPI validates from them, so there are no
      repeated `if` checks in handlers
- [ ] Invalid input returns `422` with which fields failed and why
- [ ] `model_config = ConfigDict(extra="forbid")` — unknown fields rejected, not ignored
- [ ] Separate `Create`, `Update` and `Read` models — the model that accepts input is never
      the model that returns it. That's how `is_admin` gets set by a client
- [ ] Response models declared with `response_model`, so a user's email or password hash
      can't leak through a serialisation mistake
- [ ] Post bodies have a length limit, and handles are validated against a format and a
      reserved list

**You'll learn** — parse-don't-validate, and why "the frontend already validates it" is
never a reason to skip server validation. The frontend is not the only client — `curl` is.

---

### MIRAI-020 — Consistent error handling
**Size:** M · **Depends on:** MIRAI-019

**Goal.** One error shape for the whole API, and no stack traces reaching users.

**Acceptance criteria**
- [ ] Typed error classes (`NotFoundError`, `ForbiddenError`, `RateLimitedError`, …)
- [ ] One exception handler maps them to status codes
- [ ] Response shape is always `{ "error": { "code", "message", "details"? } }`
- [ ] `500` responses expose no internals, but log everything server-side
- [ ] Every error carries a request id, and the id is in the response

**You'll learn** — status codes that mean what they say, and why leaking a stack trace
tells an attacker your framework, versions, and file paths.

---

### MIRAI-021 — Posts
**Size:** M · **Depends on:** MIRAI-020

**Goal.** Create, read, edit, and delete posts.

**Acceptance criteria**
- [ ] `POST /posts`, `GET /posts/{id}`, `PATCH`, `DELETE`
- [ ] A post targets either a community or the author's profile — never both, never neither
- [ ] Editing records `edited_at`; posts older than a cutoff can't be edited
      <!-- TODO(team): is there an edit window? -->
- [ ] Delete is soft: the post disappears from feeds but its comment thread survives
- [ ] `@mentions` parsed at write time and stored as structured references, not re-parsed
      on every read
- [ ] Bodies stored raw and escaped on output — never mangle what the user typed
- [ ] `201` on create, `204` on delete, `404` on missing

**You'll learn** — REST conventions, the storage/rendering boundary, and soft delete.

---

### MIRAI-022 — Comments and threading
**Size:** L · **Depends on:** MIRAI-021

**Goal.** Threaded replies that load efficiently at depth.

**Acceptance criteria**
- [ ] Create a comment on a post or as a reply to another comment
- [ ] `GET /posts/{id}/comments` returns a *tree*, not a flat list, with the top N branches
- [ ] Deeper branches are fetched on demand — a thread with 4,000 comments must never send
      4,000 comments
- [ ] Maximum depth enforced from MIRAI-013
- [ ] Deleting a comment with replies shows `[deleted]` and keeps the children reachable
- [ ] Replying to a comment in a community you were banned from fails
- [ ] Loading a 1,000-comment thread is a bounded number of queries — proved in the PR

**You'll learn** — serialising trees over HTTP, partial loading, and the fact that the
`[deleted]` placeholder exists in every real threaded system for a reason.

---

### MIRAI-023 — The feed
**Size:** XL · **Depends on:** MIRAI-022

**Goal.** A ranked home feed of posts from what a user follows.

**Why.** The most important ticket in the program. Every social product lives or dies on
this, and there is no correct answer — only a decision you can defend.

**Acceptance criteria**
- [ ] `GET /feed` returns ranked posts from followed users and communities
- [ ] **A written decision in the PR: fan-out on read, or fan-out on write?**
      - *On read* — query the follow graph at request time. Simple, always fresh, and gets
        slower as someone follows more accounts
      - *On write* — push each new post into every follower's feed table. Fast reads, and
        an account with 100,000 followers now causes 100,000 writes
      Pick one, implement it, and state what would make you switch
- [ ] A ranking function combining score, recency, and author affinity, written as a
      formula you can explain in one paragraph
- [ ] Ranking parameters are configuration, not magic numbers buried in code
- [ ] A user following 500 accounts gets page one in under 200ms at MIRAI-016's data scale
- [ ] Blocked users and removed posts never appear
- [ ] A new user with zero follows gets a sensible feed rather than an empty page

**You'll learn** — the defining architectural trade-off of social software, ranking
design, and the cold-start problem. The last criterion is the one people forget, and it's
the one that decides whether a new user comes back.

**Hints** — read about how the large platforms have described their own timeline
architecture. Most use a hybrid: fan-out on write for normal accounts, on read for the
accounts with enormous follower counts. Understanding *why* they split is the lesson.

---

### MIRAI-024 — Cursor pagination
**Size:** M · **Depends on:** MIRAI-023

**Goal.** Infinite scroll that stays correct while new posts arrive above you.

**Acceptance criteria**
- [ ] Cursor-based, not `OFFSET`
- [ ] The cursor encodes rank and id, so a tie in score can't skip or duplicate a post
- [ ] Response includes the next cursor and whether more exist
- [ ] `limit` capped server-side, so a client can't request a million rows
- [ ] Posting new content while a user scrolls does not shift or duplicate what they've
      already seen — and there's a test proving it
- [ ] The PR explains why offset pagination degrades and how it skips rows here specifically

**You'll learn** — cursor pagination, and the "I saw the same post twice" bug, which is
exactly what offset pagination does on a feed that's being written to.

---

### MIRAI-025 — Follows, votes, and counters
**Size:** M · **Depends on:** MIRAI-021, MIRAI-017

**Acceptance criteria**
- [ ] Follow and unfollow a user or a community; idempotent — following twice is not an error
- [ ] Vote up, vote down, change a vote, remove a vote
- [ ] Every one of those updates its denormalised counter in the same transaction
- [ ] Blocking someone removes any follow in both directions
- [ ] You cannot follow yourself, and the API says so clearly
- [ ] Rapid repeated votes on one post are rate limited and end at the correct score

**You'll learn** — idempotency, and why "follow" being safe to call twice removes an entire
class of frontend bug.

---

### MIRAI-026 — Notifications
**Size:** L · **Depends on:** MIRAI-025

**Goal.** Tell people when something involving them happens.

**Acceptance criteria**
- [ ] Notifications for: reply, mention, new follower, and your post reaching a threshold
- [ ] Generated by a **background job**, never inline in the request — a post with 500
      mentions must not slow down posting
- [ ] Idempotent: a retried job must not notify anyone twice
- [ ] Grouped — "12 people replied", not twelve rows
- [ ] Never notify someone about their own action, or about a user who blocked them
- [ ] Read / unread state, and marking all read is one query
- [ ] Paginated

**You'll learn** — fan-out, idempotency in queues, and notification grouping. The
self-notification case is the bug every social app ships at least once.

---

### MIRAI-027 — Search
**Size:** M · **Depends on:** MIRAI-024

**Acceptance criteria**
- [ ] Search posts, users, and communities — Postgres full-text, no external service
- [ ] A GIN index, with `EXPLAIN` before and after in the PR
- [ ] Results ranked by relevance combined with recency
- [ ] Removed posts, blocked users, and private communities never appear
- [ ] Empty and single-character queries handled without a 500
- [ ] Search of 500k posts returns in under 200ms

**You'll learn** — `tsvector`, `tsquery`, GIN indexes, and that Postgres handles far more
search than people assume before reaching for Elasticsearch.

---

### MIRAI-028 — API documentation
**Size:** M · **Depends on:** MIRAI-027

**Goal.** The generated OpenAPI spec made genuinely useful, and wired to the frontend type
generation from MIRAI-004.

**Acceptance criteria**
- [ ] Every endpoint has a summary, description, and tag — FastAPI gives you the schema for
      free, but not the prose
- [ ] Error responses declared with `responses={...}`, not just the happy path
- [ ] Realistic examples on request and response models
- [ ] `just gen-types` regenerates the frontend types from this spec, and CI fails if the
      committed types are stale
- [ ] A new developer can drive the whole API from `/docs` alone

**You'll learn** — API contracts, and why generated documentation is the only kind that
stays true.

---

**Milestone M1.** Demo the whole social network from the terminal: sign up two users, have
one follow the other, post, reply in a thread, vote, and read a ranked feed that reflects
all of it. No UI. If you can do that, the hard half is done.

**Next:** [Epic 04 — Auth](epic-04-auth.md)
