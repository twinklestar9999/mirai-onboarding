# Common stacks

The stack in this program is one point in a large space. This page is the map, so that a
job ad, a tech blog, or a colleague saying "we're a Rails shop" all mean something to you.

**Time:** ~20 minutes. Skim it now, come back when you meet an unfamiliar name.

---

## Why this matters

Two reasons, and the second is the important one.

First, orientation: you'll read job descriptions and conversations full of these names,
and not knowing them makes you feel further behind than you are.

Second — and this is the actual lesson — **the same shapes recur everywhere.** Every one
of these stacks has a router, a way to validate input, a way to talk to a database, a way
to run background work, and a way to render UI. Once you've learned those *roles* properly
in one stack, learning another is mostly renaming. Your first stack takes months. Your
third takes a fortnight.

That's why this program goes deep on one stack rather than skimming five.

---

## Backend stacks

### Python

| Stack | Shape | You'll see it in |
| --- | --- | --- |
| **FastAPI** (ours) | Async, type-hint driven, you assemble the pieces | New APIs, ML serving, microservices |
| **Django** | Batteries included: ORM, admin, auth, templates, all decided for you | Established products, content-heavy sites, teams that value convention |
| **Flask** | Minimal core, you pick every extension | Small services, older codebases, internal tools |

**Django is the one to learn second.** It's the most common Python job, and coming to it
after FastAPI is instructive: you'll recognise every piece it hands you, because you built
each one yourself. The admin panel alone explains why teams choose it.

### JavaScript / TypeScript

| Stack | Shape |
| --- | --- |
| **Express** | The old default. Minimal, unopinionated, everywhere |
| **NestJS** | Angular-style structure, decorators, dependency injection. Enterprise-shaped |
| **Hono / Fastify** | Modern, fast, small. Hono runs on edge runtimes |
| **Next.js API routes** | Backend living inside the frontend framework |

### Elsewhere

| Stack | Language | Known for |
| --- | --- | --- |
| **Ruby on Rails** | Ruby | The original "convention over configuration." Extremely fast to build in |
| **Spring Boot** | Java | Enterprise default. Verbose, mature, everywhere in banking and large corporates |
| **Laravel** | PHP | Rails' ideas, done well. Still a very large share of the web |
| **ASP.NET Core** | C# | Microsoft ecosystem, strong tooling, common in corporate settings |
| **Go** (Gin, Echo, stdlib) | Go | Infrastructure, high-concurrency services. Tiny deployable binaries |
| **Phoenix** | Elixir | Real-time and websockets at scale. Small but devoted following |

---

## Frontend stacks

| Stack | Shape | When |
| --- | --- | --- |
| **React + Vite** (ours) | SPA, you choose routing and data fetching | Apps behind a login, where SEO doesn't matter |
| **Next.js** | React with server rendering, routing and API routes included | Content that needs SEO; teams wanting one framework for everything |
| **Vue + Nuxt** | Vue's equivalent pairing. Gentler learning curve than React | Common in Europe and Asia |
| **Angular** | Full framework: DI, RxJS, strong conventions | Large enterprise apps, long-lived teams |
| **Svelte / SvelteKit** | Compiles away; very little runtime | Smaller bundles, developer happiness |
| **HTMX + server templates** | No SPA at all — the server sends HTML | A real and growing reaction to SPA complexity. Pairs with Django, Rails, Laravel |

**On SPAs and SSR:** we chose a plain SPA deliberately, because it keeps the boundary
between client and server visible while you're still learning where it is. Next.js blurs
that boundary on purpose, which is powerful once you understand it and confusing before.

---

## The named combinations

You'll hear these as single words:

- **LAMP** — Linux, Apache, MySQL, PHP. The stack most of the early web ran on.
- **MEAN / MERN** — MongoDB, Express, Angular or React, Node. The 2015 JavaScript default.
  Notable mostly for MongoDB, which was frequently the wrong choice for data that was
  obviously relational.
- **JAMstack** — Prebuilt static pages plus APIs. Fast and cheap; awkward once content
  gets personalised.
- **T3** — Next.js, TypeScript, tRPC, Prisma, Tailwind. Type safety end to end, without a
  codegen step.
- **PERN** — Postgres, Express, React, Node. MERN with a sensible database.

---

## Databases

| Kind | Examples | Use when |
| --- | --- | --- |
| **Relational** | PostgreSQL, MySQL, SQLite | Your data has relationships. This is nearly always the right default |
| **Document** | MongoDB, DynamoDB | Genuinely schemaless data, or a known access pattern at extreme scale |
| **Key–value** | Redis, Valkey | Caching, sessions, queues, rate limits |
| **Search** | Elasticsearch, OpenSearch, Typesense | Search beyond what Postgres full-text handles |
| **Analytical** | ClickHouse, BigQuery, Snowflake | Aggregating billions of rows for reporting |

**A caution worth internalising early:** the most common database mistake juniors inherit
is a document store holding data that is plainly relational — users, orders, and items,
joined by hand in application code. Postgres handles JSON perfectly well when you need it.
Start relational; move off it when something specific forces you to.

---

## Hosting

| Kind | Examples | Trade |
| --- | --- | --- |
| **PaaS** | Vercel, Netlify, Railway, Render, Fly.io, Heroku | Fast and pleasant; less control, and cost scales with usage |
| **Managed containers** | ECS Fargate, Cloud Run | Middle ground. You own the image, they own the machines |
| **Orchestrated** | Kubernetes (EKS, GKE) | Maximum control and maximum operational cost. Needs a team |
| **Serverless functions** | Lambda, Cloud Functions | Cheap when idle; awkward for long tasks and database connections |
| **Plain servers** | A VPS, bare metal | Cheapest at scale, and everything is your problem |

You'll touch three of these directly: managed containers in [epic 08](epic-08-aws.md) and
[epic 09](epic-09-gcp.md), and a PaaS in [epic 10](epic-10-vercel.md).

---

## How to read a job ad

Given a stack you've never used, ask these five questions. The answers usually take ten
minutes to find, and they tell you almost everything:

1. **What routes a request?** Express, Django URLs, Rails routes, FastAPI decorators.
2. **What validates input?** Pydantic, Zod, Rails strong parameters, Java Bean Validation.
3. **How does it reach the database?** SQLAlchemy, Django ORM, ActiveRecord, Hibernate, raw SQL.
4. **How does it run work later?** Celery, Sidekiq, BullMQ, a cloud queue.
5. **How does it render UI?** Server templates, a SPA, or a hybrid.

Every stack answers all five. Once you can locate the answers, you can read the codebase.

---

## What to do with this

**Don't go and learn them.** Learning three stacks shallowly is the most common way early
developers spend a year and end up unable to build anything alone.

Finish this program. Get genuinely good at one stack — deep enough that you understand
*why* each piece exists, which is what epics 02–06 are actually for. Then a second stack
is a fortnight, and you'll be able to tell which of its choices are better and which are
just different.

That judgement is the thing that's hard to acquire, and it doesn't come from breadth.

---

**Next:** [The project](03-project-spec.md) — what you're actually building
