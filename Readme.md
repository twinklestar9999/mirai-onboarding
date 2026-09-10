# Mirai Onboarding

A structured program that takes a junior developer from "I can write code" to "I can
ship and run a product."

You do it by building one real, public website — **Mirai**, a social site built around
communities — across **101 tickets in 11 epics**, covering everything from schema design
to running it in production on two clouds.

At the end you have a URL you can send someone. They sign up, join a community, post an
image, and reply in a thread — on their phone, quickly, without it falling over. That is
the goal, and every epic is scoped by whether it moves you toward it.

It's written for developers early in their career, working remotely. It assumes you can
write code and assumes nothing else: not that you've used a professional git workflow,
not that you've had a pull request reviewed, not that you know what half the words in
your first standup mean.

---

## Start here

Read these, in order. About two and a half hours total.

| | | |
| --- | --- | --- |
| 1 | [The stack](docs/01-stack.md) | Every tool you'll learn, and why it was chosen over the alternative |
| — | [Common stacks](docs/02-common-stacks.md) | The map our stack sits in — skim now, return later |
| 2 | [The product](docs/03-project-spec.md) | Mirai — the site you're building, screen by screen, and the demo you finish with |
| 3 | [Setup](docs/04-setup.md) | Get the toolchain working on your machine |
| 4 | [Asking for help](docs/06-asking-for-help.md) | Read this **before** you get stuck, not after |
| 5 | [Using Claude](docs/07-using-claude.md) | How to use an AI assistant without hollowing out your own learning |

Then open [the ticket board](docs/12-ticket-board.md) and start at MIRAI-001.

---

## The program

| Epic | Tickets | What you learn |
| --- | --- | --- |
| [01 — Foundation](docs/epic-01-foundation.md) | 001–008 | Repo layout, strict typing, the Python↔TypeScript contract, first CI |
| [02 — Database](docs/epic-02-database.md) | 009–017 | The social graph, comment trees, SQLAlchemy, Alembic, real SQL, indexes at scale |
| [03 — API](docs/epic-03-api.md) | 018–028 | FastAPI, Pydantic, threaded comments, **the feed**, cursor pagination, search |
| [04 — Auth](docs/epic-04-auth.md) | 029–041 | Sessions and RBAC by hand, then OIDC, social login, third-party APIs, scoped tokens |
| [05 — Frontend](docs/epic-05-frontend.md) | 042–055 | React, infinite feed, composer, threaded comments, uploads, accessibility |
| [06 — Testing](docs/epic-06-testing.md) | 056–062 | pytest, Testcontainers, Vitest, Playwright |
| [07 — Containers & CI/CD](docs/epic-07-containers-ci.md) | 063–069 | Docker, Compose, pipelines, releases, rollback |
| [08 — AWS](docs/epic-08-aws.md) | 070–079 | Terraform, ECS, RDS, S3, CloudFront, IAM |
| [09 — GCP](docs/epic-09-gcp.md) | 080–086 | Cloud Run, Cloud SQL, GCS — the same app, a second vendor |
| [10 — Vercel](docs/epic-10-vercel.md) | 087–092 | A platform instead of a cloud, and an honest three-way comparison |
| [11 — Production](docs/epic-11-production.md) | 093–101 | Observability, load testing, security, backups, incident drill |

**Five to seven months** at a normal pace. That's the honest estimate, not a challenge.

### Milestones

| | After | You can demo |
| --- | --- | --- |
| **M1** | Epic 03 | A working API, drivable entirely from `curl`. No UI. |
| **M2** | Epic 05 | Sign up, post, reply in a thread, scroll a real feed. In a browser. |
| **M3** | Epic 07 | Tests catch a bug you planted; CI blocks a bad PR. |
| **M4** | Epic 10 | The same app live on two clouds and a platform, at real URLs. |

---

## How we work

Reference material. Read [asking for help](docs/06-asking-for-help.md) early; read the
rest when a ticket needs it.

| | |
| --- | --- |
| [How we work](docs/05-how-we-work.md) | Async communication, working hours, standup, 1:1s |
| [Using Claude](docs/07-using-claude.md) | Where it helps, where it quietly hurts, and the rules here |
| [Design with Claude](docs/08-design-with-claude.md) | Sketching a screen before you build it — for epic 05 |
| [Asking for help](docs/06-asking-for-help.md) | The 30-minute rule, and how to ask so you get answered |
| [Git workflow](docs/09-git-workflow.md) | Branches, commits, pull requests, getting out of trouble |
| [Code review](docs/10-code-review.md) | Getting reviewed, and reviewing others |
| [Glossary](docs/11-glossary.md) | Every term we use that nobody explains |

---

## What we expect of you

Nobody expects you to be fast. Here's what we actually watch for:

- **You ask early.** Being stuck for two hours is normal. Being stuck for two days
  without telling anyone is the only real mistake available to you right now.
- **You leave a trail.** Your work is visible in tickets, branches and PRs — not just in
  your head. Remote teams run on what's written down.
- **You read the error.** Most of what blocks a junior developer is already printed on
  the screen.
- **You repeat a mistake at most once.** Nobody minds the first time.

What we do *not* expect: knowing the stack already, working outside your hours, having
opinions about architecture yet, or getting a PR approved on the first pass. First PRs
get comments. That's the process working, not you failing.

---

## Something here is wrong or missing?

Fix it. This guide is a repo like any other — open a pull request. The person best
placed to spot a gap in onboarding is the person being onboarded, and that window closes
after about three weeks.

Placeholders marked as team TODOs are facts nobody has filled in yet — repo URLs, chat
channels, cloud account details. If one is blocking you, ask rather than guess.
