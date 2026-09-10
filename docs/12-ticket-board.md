# The ticket board

The whole project, broken into tickets. Work them in order — dependencies mostly run
top to bottom.

**Ticket IDs are `MIRAI-NNN`.** Use the ID in your branch name, commit messages, and PR
title so everything about a piece of work links together:

```bash
git checkout -b feat/MIRAI-022-create-task-endpoint
git commit -m "MIRAI-022: add POST /tasks with Pydantic validation"
```

---

## Sizes

| Size | Roughly | If it takes much longer |
| --- | --- | --- |
| **S** | 2–4 hours | Normal on your first few. Say so in standup. |
| **M** | ~1 day | Fine. Post an update at the end of day one. |
| **L** | 2–3 days | Post progress daily. |
| **XL** | ~1 week | Should have been split. Ask about splitting it. |

These are how long they take *someone who already knows the stack*. Your first pass at
an S ticket taking two days is expected and is not a problem — a silent two days is.

---

## Epics

| # | Epic | Tickets | Focus |
| --- | --- | --- | --- |
| 01 | [Foundation](epic-01-foundation.md) | MIRAI-001 – 008 | Repo layout, strict typing, tooling, the Python↔TS contract, first CI |
| 02 | [Database](epic-02-database.md) | MIRAI-009 – 017 | The social graph, comment trees, SQLAlchemy, Alembic, indexes at scale |
| 03 | [API](epic-03-api.md) | MIRAI-018 – 028 | FastAPI, Pydantic, threaded comments, the feed, cursor pagination |
| 04 | [Auth](epic-04-auth.md) | MIRAI-029 – 041 | Sessions, hashing, communities and moderation, OIDC, OAuth three ways |
| 05 | [Frontend](epic-05-frontend.md) | MIRAI-042 – 055 | React, infinite feed, composer, threaded comments, uploads |
| 06 | [Testing](epic-06-testing.md) | MIRAI-056 – 062 | pytest, Testcontainers, Vitest, MSW, Playwright |
| 07 | [Containers & CI/CD](epic-07-containers-ci.md) | MIRAI-063 – 069 | Docker, Compose, GitHub Actions, releases |
| 08 | [AWS](epic-08-aws.md) | MIRAI-070 – 079 | Terraform, ECS, RDS, S3, CloudFront, IAM |
| 09 | [GCP](epic-09-gcp.md) | MIRAI-080 – 086 | Cloud Run, Cloud SQL, GCS — the same app, a second vendor |
| 10 | [Vercel](epic-10-vercel.md) | MIRAI-087 – 092 | A platform instead of a cloud — and what it hides |
| 11 | [Production](epic-11-production.md) | MIRAI-093 – 101 | Observability, performance, security, incident drill |

**101 tickets.** Expect five to seven months. That is the honest number, not a challenge.

---

## Dependency map

```
01 Foundation
     │
     ├──> 02 Database ──> 03 API ──> 04 Auth ──┐
     │                        │                │
     │                        └──> 05 Frontend ┘
     │                                  │
     └──────────────────> 06 Testing <──┘
                               │
                               v
                        07 Containers & CI
                               │
                  ┌────────────┼────────────┐
                  v            v            v
               08 AWS       09 GCP     10 Vercel
                  └────────────┼────────────┘
                               v
                        11 Production
```

Epics 08, 09 and 10 can be done in any order, though doing Vercel last makes its
comparison ticket much sharper. Everything else is sequential — a ticket whose
dependency isn't merged will waste your time.

---

## Milestones

Four points where you have something real to show. Demo each one to your buddy or your
manager — explaining your own work out loud is where most of the learning consolidates.

| Milestone | After | You can demo |
| --- | --- | --- |
| **M1 — It stores things** | Epic 03 | A working social network you can drive with `curl`. No UI yet. |
| **M2 — It's a product** | Epic 05 | Sign up, post, reply in a thread, scroll a real feed. |
| **M3 — It's trustworthy** | Epic 07 | Tests catch a bug you deliberately introduce; CI blocks a bad PR. |
| **M4 — It's live** | Epic 10 | The same app on two clouds and a platform, at real URLs. |

---

## Ticket format

Every ticket looks like this:

> ### MIRAI-000 — Short imperative title
> **Size:** M · **Depends on:** MIRAI-000
>
> **Goal.** What must be true when this is done.
>
> **Why.** What this teaches, or what breaks without it.
>
> **Acceptance criteria** — the checklist a reviewer will actually check.
>
> **You'll learn** — the concepts to go read about.
>
> **Hints** — where to look when you're stuck, *after* you've tried.

Read **Goal** and **Acceptance criteria** before you start. Read **Hints** only after
you've been stuck for a while — the struggle before the hint is where the learning is.

---

## Before your first ticket

- [ ] [Stack](01-stack.md) and [common stacks](02-common-stacks.md) read
- [ ] [Project spec](03-project-spec.md) read
- [ ] [Setup](04-setup.md) complete — the app runs locally
- [ ] [Git workflow](09-git-workflow.md) read
- [ ] You know the 30-minute rule from [asking for help](06-asking-for-help.md)
- [ ] [Using Claude](07-using-claude.md) read — especially the rules section

---

**Next:** [Epic 01 — Foundation](epic-01-foundation.md) — start at MIRAI-001
