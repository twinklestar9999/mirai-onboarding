# Glossary

Every term the team drops into conversation without explaining. Nobody is hiding these
from you — they've just forgotten there was a time they didn't know them.

If a word confuses you and it isn't here, **add it**. That's a genuinely useful PR.

---

## Git and code review

**main / master** — The primary branch. What's on it is meant to always work.

**branch** — Your own copy of the code to work on without disturbing anyone else.

**commit** — A saved snapshot of your changes, with a message explaining them.

**PR / pull request / MR** — A request to merge your branch into `main`, plus the
discussion about it. "MR" (merge request) is the same thing under a different name.

**diff** — The lines added and removed by a change. What a reviewer reads.

**merge** — Combining your branch into another. **merge conflict** — git can't tell
which of two versions of a line to keep, so you choose.

**rebase** — Replaying your commits on top of a newer `main` for a straighter history.
Powerful, and easy to lose work with. Ask before rebasing a pushed branch.

**force push** — Overwriting the remote branch's history. Can delete other people's
work. Ask first, always.

**squash** — Combining several commits into one, usually when merging.

**cherry-pick** — Copying one specific commit onto another branch.

**revert** — A new commit that undoes an old one. The safe way to take something back.

**LGTM** — "Looks good to me" — approval. **nit** — a minor, non-blocking suggestion.
**WIP** — work in progress, not ready for review.

**CODEOWNERS** — A file saying who must review changes to which parts of the code.

## Environments and shipping

**local** — On your machine. **staging** — A production-like environment for testing
before release. **prod / production** — The real thing, with real users.

**deploy / ship / release** — Getting code out to an environment.

**rollback** — Putting the previous version back after a bad deploy.

**CI** — Continuous integration: the automation that runs your tests on every push.
"CI is red" means something failed. **CD** — the deploy half of the same pipeline.

**the pipeline** — The chain of automated steps from push to deployed.

**feature flag** — A switch that turns a feature on or off without deploying, so code
can ship dark and be enabled later.

**hotfix** — An urgent fix going straight to production outside the normal process.

**incident** — Something is broken for users right now. **postmortem** — the written
account of what happened afterwards. These are blameless: the goal is fixing the
system, never assigning fault to a person.

**on call** — Whoever is responsible for responding if something breaks right now.

## Work and process

**ticket / issue / story** — A unit of work in the tracker.

**backlog** — Work identified but not scheduled.

**sprint** — A fixed time block (often two weeks) of planned work.

**standup** — The short daily sync: done / doing / blocked.

**retro** — Retrospective: the team reviewing how the last stretch went.

**blocked** — You can't proceed without something or someone else. Say it early.

**scope creep** — A task quietly growing beyond what was agreed.

**spike** — Time-boxed investigation to answer a question before committing to work.

**estimate** — A guess at effort. Everyone's are wrong; say when yours turns out to be.

**1:1** — A recurring private meeting with your manager. Yours to steer.

**async** — Communication that doesn't need both people present. The default here.

**EOD / EOW** — End of day / end of week. Ambiguous across timezones — ask whose.

**PTO / OOO** — Time off / out of office.

## Code and quality

**refactor** — Changing code's structure without changing what it does.

**tech debt** — A shortcut taken deliberately that will cost time later. Normal in
moderation; the problem is untracked debt.

**regression** — Something that used to work and now doesn't.

**edge case** — An unusual input or state: empty, zero, null, enormous, simultaneous.
Where most bugs live.

**flaky test** — A test that passes and fails without the code changing. Not your fault
if you hit one; say so rather than re-running until it's green.

**unit test / integration test / e2e** — Testing one piece in isolation / several
pieces together / the whole system as a user would.

**mock / stub / fixture** — A stand-in for a real dependency in a test, and canned data
for it to use.

**linter** — A tool flagging style and likely-bug patterns automatically.

**breaking change** — A change that makes existing callers stop working.

**backwards compatible** — Old callers keep working.

**deprecated** — Still works, but on its way out. Don't build new things on it.

**boilerplate** — Repetitive code required to make something work.

**race condition** — A bug where the outcome depends on which of two things finishes
first. Intermittent and irritating to reproduce.

**idempotent** — Safe to do twice. Running it again changes nothing further.

## Web and systems

**API** — An interface one program uses to talk to another.

**endpoint** — One specific URL an API exposes.

**request / response** — What the client sends, and what the server sends back.

**payload / body** — The data carried in a request or response.

**status code** — The number in a response. 2xx worked, 4xx the caller's fault,
5xx the server's fault.

**auth** — Ambiguous shorthand for two things: **authentication** (who are you) and
**authorization** (what are you allowed to do). Worth asking which is meant.

**token** — A string proving you're authenticated. Always a secret.

**migration** — A scripted change to the database's structure.

**cache** — A stored copy of something expensive to compute, to serve it faster.
Also the first suspect when someone sees stale data.

**env var** — Configuration passed in from outside the code, so the same code can run
in different environments.

**latency** — How long something takes. **throughput** — how much it handles per second.

## Python

**venv / virtual environment** — An isolated set of packages per project, so two
projects can need different versions of the same library. `uv` manages one for you.

**lockfile** — `uv.lock` / `pnpm-lock.yaml`: the exact versions actually installed.
Committed, so everyone and CI get identical dependencies.

**type hints** — `def f(x: int) -> str`. Python ignores them at runtime; **mypy** is
what checks them. Hints without mypy are documentation, not safety.

**strict mode** — mypy configuration that rejects untyped code. On from day one here.

**ASGI** — The async server interface FastAPI speaks. **WSGI** is its older, synchronous
predecessor. **uvicorn** is the ASGI server you run in development.

**async / await** — Code that can pause while waiting on I/O so the process handles other
requests meanwhile. **blocking** — a call that doesn't pause, and stalls everything else
on that worker. Mixing a blocking call into async code is a classic Python performance bug.

**coroutine** — What an `async def` function returns. It does nothing until awaited.

**Pydantic model** — A class that validates and coerces data from its type hints. The
backend's contract with the outside world.

**ORM** — Object-relational mapper: **SQLAlchemy** here. Maps rows to Python objects.

**session** (SQLAlchemy) — The unit of work holding pending changes until you commit.
Unrelated to a login session, confusingly.

**lazy loading** — An ORM fetching a relationship only when you touch it — which is how
you accidentally run one query per row. See **N+1**.

**eager loading** — Fetching relationships up front (`selectinload`), in one query.

**Alembic revision** — One migration file: a schema change, forward and back.

**fixture** (pytest) — Reusable setup for tests, requested by naming it as an argument.

**parametrize** — Running one test over many inputs instead of copy-pasting it.

**Celery task** — A function run later, by a worker, off the request path.

**GIL** — The global interpreter lock: only one thread runs Python bytecode at a time.
Why Python scales with processes and async I/O rather than threads.

## Cloud and infrastructure

**IaC** — Infrastructure as code. Your servers defined in files, reviewed and versioned
like any other code. **Terraform** here.

**state** (Terraform) — Terraform's record of what it created. Kept remotely so a team
can share it, and locked so two people can't apply at once.

**plan / apply** — `plan` shows what would change; `apply` does it. You always read the
plan. It is a code review for your infrastructure.

**drift** — Reality no longer matching the code, usually because someone clicked
something in the console.

**region / availability zone** — Where your resources physically live. Spreading across
zones is how you survive one datacenter failing.

**VPC** — Your private network in the cloud. **subnet** — a slice of it; **public**
subnets can reach the internet, **private** ones can't.

**security group** — A firewall around a resource, saying what may reach it.

**IAM** — Who may do what to which resource. **least privilege** — grant only what's
needed, which is harder and correct.

**service account** — A non-human identity used by an application.

**OIDC federation / workload identity** — Letting CI authenticate to a cloud without a
stored key. Preferred, because a key in a file is a key that can leak.

**container image** — A packaged filesystem and process. **registry** — where images are
stored (ECR, Artifact Registry).

**Fargate / Cloud Run** — Run a container without managing servers.

**cold start** — The delay when a scaled-to-zero service handles its first request.

**CDN** — Cached copies of your assets near users. **invalidation** — telling it to
forget a cached file after a deploy.

**presigned / signed URL** — A time-limited URL letting a browser upload to or download
from private storage directly, without the file passing through your API.

**blue/green, canary** — Deploy strategies: run the new version alongside the old, or
send it a small share of traffic first.

**IaaS / PaaS / managed service** — Increasing amounts of the operating done for you, at
increasing cost and decreasing control.

## Operations

**observability** — Being able to answer what happened, from outside the system.

**structured logging** — Logs as JSON objects you can query, not prose you scroll.

**trace / span** — One request's journey through the system, and each step within it.

**correlation / request id** — The id tying one request's logs, traces and errors
together. What makes "it broke at 3pm" solvable.

**p50 / p95 / p99** — Latency percentiles. p99 is the experience of your unluckiest 1%,
and averages hide them entirely.

**golden signals** — Latency, traffic, errors, saturation. Monitor these first.

**SLO / SLI / error budget** — The target, the measurement, and how much failure you've
agreed you can spend before you stop shipping features.

**runbook** — Step-by-step instructions for handling a specific alert, written for
someone tired.

**RPO / RTO** — How much data you can afford to lose, and how long you can afford to be
down. Both are business decisions, not technical ones.

**game day** — A deliberately triggered incident, to practise responding.

**blameless postmortem** — The write-up after an incident. Its output is a changed
system, never a chastened person.

**toil** — Manual repetitive operational work. The thing automation is for.

## OAuth and identity

**authentication** — Who are you. **authorization** — What you're allowed to do. Two
different things that get shortened to the same word, "auth". Worth asking which is meant.

**OAuth 2.0** — A protocol for granting an application limited access to your account
somewhere else, without giving it your password. It is about *authorization*, despite
being what "Sign in with Google" runs on.

**OIDC** (OpenID Connect) — A thin identity layer on top of OAuth 2.0. This is the part
that actually says who you are. If you want login, you want OIDC, not bare OAuth.

**authorization code flow** — The standard flow: the app sends you to the provider, the
provider sends back a short-lived *code*, and the app exchanges that code for tokens from
its own server. The exchange happens server-side so the tokens never touch the browser.

**PKCE** — An extension that stops an attacker who intercepts the code from exchanging it,
by requiring proof that the exchange comes from whoever started the flow. Originally for
mobile apps; now recommended everywhere.

**state parameter** — A random value you send out and verify on the way back. It's what
proves the callback belongs to a flow *you* started, and skipping it leaves login open to
CSRF. The most commonly omitted step in OAuth implementations.

**redirect URI** — Where the provider sends the user back. Must be matched exactly against
an allowlist; loose matching is a standard way accounts get taken over.

**access token** — Short-lived credential presented with each API call.
**refresh token** — Longer-lived credential used to get a new access token. More dangerous
if leaked, so it's stored encrypted and never sent to a browser.

**scope** — What a token is permitted to do. **incremental authorization** — asking for a
scope when the user first needs it, rather than everything at signup.

**consent screen** — Where the provider asks the user to approve the scopes you requested.

**bearer token** — Any token where possession alone is sufficient. Anyone holding it is
you, which is why it never goes in a URL, a log, or a git repo.

**PAT / personal access token** — A long-lived token a user generates for scripts and CI.

**account linking** — Connecting a social login to an existing account. Safe only when the
provider asserts the email is verified.

**SSO** — One login across many applications. **SAML** — the older enterprise protocol
that does this; you'll meet it in corporate integrations.

**MFA / 2FA** — Requiring a second factor beyond a password.

**JWT** — A signed, self-contained token. Convenient, and hard to revoke before it
expires — which is why this program uses server-side sessions instead.

## Deployment platforms

**PaaS** — Platform as a service: you push code, it handles servers, TLS, CDN and scaling.
Vercel, Netlify, Railway, Render, Fly.io, Heroku.

**preview deployment** — A live URL per pull request, so reviewers can click instead of
imagine. Vercel's best feature, and worth reproducing anywhere.

**edge function** — Code running at a CDN location near the user rather than at one origin
region. Fast, and constrained: short execution limits, no native modules, no direct
database connections.

**cold start** — The delay when a scaled-to-zero service handles its first request.

**Core Web Vitals** — Google's user-experience metrics: **LCP** (how long until the main
content appears), **INP** (how quickly the page responds to input), **CLS** (how much the
layout jumps around). Measured on real users, not just in a lab.

**RUM** — Real user monitoring: metrics from actual visitors. **synthetic** — metrics from
a scripted test. You need both; they disagree, and the disagreement is informative.

**lock-in** — How hard it would be to leave. Only meaningful as a number: how many weeks,
and what would you have to rebuild.

## AI-assisted development

**LLM** — Large language model. The thing behind Claude and its equivalents.

**context** — What the model can currently see: your files, the conversation, the error you
pasted. Most bad answers are missing context rather than missing capability.

**prompt** — What you ask. **system prompt** — standing instructions that shape every
answer in a session.

**hallucination** — A confident, fluent, wrong answer. Most common on version numbers,
library APIs, and anything that changed recently. Indistinguishable from a right answer
by tone alone, which is why you verify rather than vibe-check.

**agent** — A model that can run tools — read files, run commands, edit code — rather than
only producing text.

**MCP** — A standard protocol for connecting tools and data sources to an AI assistant.

**skill / slash command** — Packaged instructions for a recurring task, invoked by name.

<!-- TODO(team): add terms specific to this product and codebase — internal service
     names, domain vocabulary, acronyms used in tickets -->

---

**Next:** [The ticket board](12-ticket-board.md) — the program itself
