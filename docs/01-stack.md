# The stack

Everything you'll learn on this program, and why each piece is here rather than an
alternative. Read this once now; come back to it whenever a ticket introduces something
new.

**Time:** ~20 minutes

---

## The principle behind these choices

Three rules picked this list:

1. **Python on the backend, TypeScript on the frontend.** Two languages, each the
   strongest choice for its side, connected by a generated contract so they can't drift
   apart.
2. **Boring and employable over new and clever.** Every tool here is something teams
   actually hire for. Where a newer tool is better but rarer, we chose the common one.
3. **Nothing that hides the fundamental.** You'll write SQL, not just call an ORM
   method. You'll build a session before you use an auth provider. The abstraction is
   easier to learn *after* the thing it abstracts.

---

## Backend — Python

| Layer | Choice | Why this, not the alternative |
| --- | --- | --- |
| Language | **Python 3.13** | Type hints throughout. Huge ecosystem, and the language you'll meet in data, ML, scripting and backend work alike. |
| Framework | **FastAPI** | Async, and it derives request validation *and* your OpenAPI spec from the same type hints. Django is heavier and hides more; Flask makes you assemble the same pieces by hand. |
| Validation | **Pydantic v2** | One model validates the request, types the handler, and generates the schema the frontend consumes. |
| Server | **uvicorn + gunicorn** | ASGI in development, process management in production. |
| Package manager | **uv** | Fast, lockfile-based, manages the Python version too. Replaces pip + venv + pip-tools in one tool. |
| ORM | **SQLAlchemy 2.0** (async) | The standard. Its 2.0 style is explicit and SQL-shaped, so you learn what your queries actually do. |
| Migrations | **Alembic** | Schema changes as reviewable, replayable files. |
| Background jobs | **Celery + Redis** | The queue you'll meet in real Python codebases. The moment you send an email you need one. |
| Type checking | **mypy --strict** | Python's type hints do nothing at runtime. mypy is what makes them real. Strict from day one. |
| Lint & format | **Ruff** | Lint and format in one very fast tool. Replaces flake8 + black + isort. |
| Testing | **pytest** | Fixtures and parametrize are the two features that make Python testing pleasant. |
| Auth | **Sessions by hand → OIDC → OAuth** | You build cookie sessions and Argon2 hashing yourself, replace them with a provider, then do OAuth in all three directions: as a client (social login), as a consumer of someone else's API, and as an issuer of scoped tokens. Auth you don't understand is auth you can't debug — and you *will* be asked to debug it. |

## Frontend — TypeScript

| Layer | Choice | Why |
| --- | --- | --- |
| Language | **TypeScript 5.x** | Types catch a whole class of bug before it runs — invaluable when you're new enough not to spot it by eye. |
| UI | **React 19** | The default of the industry. What you learn transfers to most jobs you'll apply for. |
| Build | **Vite** | Instant dev server, no config to fight. |
| Routing | **React Router v7** | The most widely used React router. |
| Server state | **TanStack Query** | Caching, refetching and loading states are most of frontend work. You'll do it by hand with `useEffect` once, in a ticket, to learn why this exists. |
| Forms | **React Hook Form + Zod** | Zod mirrors the Pydantic rules on the client for instant feedback. The server still validates — always. |
| Styling | **Tailwind CSS v4** | Styling without inventing class names or leaving your markup. |
| Components | **shadcn/ui** (Radix) | You own the component code, so you can read it and change it. Accessibility correct by default — worth studying. |
| Package manager | **pnpm** | Strict by default: it won't let you import a package you didn't declare. |

## The contract between them

This is the piece that replaces "one language everywhere", and it's worth understanding
properly:

```
Pydantic models  ──>  FastAPI generates OpenAPI  ──>  openapi-typescript  ──>  TS types
   (backend)              (automatic)                  (codegen step)         (frontend)
```

You define a `Task` once, in Python. FastAPI turns it into an OpenAPI schema for free.
A generator turns that schema into TypeScript types the frontend imports. Rename a field
in Python, regenerate, and the frontend **fails to compile** until it's fixed.

CI regenerates and fails if the committed types are out of date, so the two sides cannot
silently drift. That check is set up in MIRAI-008.

## Testing

| Layer | Choice | Why |
| --- | --- | --- |
| Backend unit / integration | **pytest + pytest-asyncio** | Fixtures make real setup and teardown manageable. |
| Database tests | **Testcontainers** | Tests run against a real Postgres in Docker, not a mock. Mocked databases pass while production breaks. |
| API tests | **httpx** `AsyncClient` | Drives the app through real ASGI, not by calling functions. |
| Frontend components | **Vitest + Testing Library** | Tests what a user does, not internal state. |
| API mocking | **MSW** | Intercepts at the network layer, so real fetch code runs. |
| End-to-end | **Playwright** | Drives a real browser. Your safety net before every deploy. |

## Database

**PostgreSQL 17** — the default serious relational database. Everything you learn is
standard SQL. Plus **Redis** for the queue, caching, and rate limiting.

## Infrastructure and cloud

This is the part most juniors never get to touch. It's also the part that turns you from
"can build a feature" into "can ship a product."

| Layer | Choice | Why |
| --- | --- | --- |
| Containers | **Docker + Compose** | One command to get Postgres and Redis running locally, and the same image runs in the cloud. |
| Infrastructure as code | **Terraform** | **The key choice on this program.** The one tool that provisions both AWS and GCP, so you learn cloud *concepts* once and then see two vendors' names for the same idea. Clicking around a web console teaches you nothing repeatable. |
| CI/CD | **GitHub Actions** | Tests on every PR, deploy on every merge. |
| Secrets | **AWS Secrets Manager / GCP Secret Manager** | Because secrets never, ever go in the repo. |
| Observability | **OpenTelemetry + Sentry** | Vendor-neutral tracing, and errors that find you before users do. |
| Platform | **Vercel** | Deployed to last, on purpose. Once you've built CDN, TLS, CI and preview environments by hand, a platform doing it in twenty minutes is a lesson rather than a shortcut. |

### AWS, GCP, and a platform

You deploy the same application three times — AWS, then GCP, then Vercel. That's
deliberate: seeing two vendors solve the same problem is what separates cloud
understanding from memorised button locations. Seeing a platform do it for you
*afterwards* is what lets you say precisely what it's doing on your behalf and what it
charges for that.

The order matters. A platform first teaches you a dashboard; a platform last teaches you
what the dashboard replaces.

| The concept | AWS | GCP |
| --- | --- | --- |
| Run a container | ECS Fargate | Cloud Run |
| Managed Postgres | RDS | Cloud SQL |
| Object storage | S3 | Cloud Storage |
| CDN | CloudFront | Cloud CDN |
| Load balancing | ALB | Cloud Load Balancing |
| Secrets | Secrets Manager | Secret Manager |
| Logs and metrics | CloudWatch | Cloud Logging / Monitoring |
| Permissions | IAM | IAM |
| Private networking | VPC | VPC |
| Email | SES | SendGrid |

By the end you should be able to read that table in either direction and know what the
service actually *does* — not just its name.

> **Cost warning.** Cloud spend is real money and easy to leave running by accident.
> Before the first cloud ticket you set up budget alerts and a hard spending cap, and
> every cloud epic ends with a teardown ticket. Read the cost note in
> [epic 08](epic-08-aws.md) before you provision anything.
> <!-- TODO(team): who owns the cloud accounts, what's the per-junior budget cap, and
>      is it a sandbox account or a shared one? -->

## AI assistance

You'll use Claude throughout, and that's expected rather than tolerated. Read
[using Claude](07-using-claude.md) before ticket one — particularly the tickets marked
*by hand*, where handing the work to an assistant skips the entire point of the ticket.

## Code quality

| Tool | Purpose |
| --- | --- |
| **Ruff** | Python lint and format, one fast tool. |
| **mypy --strict** | Python type checking. Without it, type hints are decoration. |
| **Biome** | TypeScript lint and format on the frontend. |
| **TypeScript strict mode** | On from day one. Turning it on later is agony. |
| **pre-commit** | Runs all of the above before a commit, so CI rarely catches style. |
| **Conventional Commits** | Readable history, and automatic changelogs later. |

---

## What you are *not* learning here, and why

Being explicit so you don't feel behind for missing them:

- **Django.** FastAPI keeps the request/response cycle visible while you're still
  learning where it is. Django's conveniences are easier to appreciate afterwards.
- **Kubernetes.** Fargate and Cloud Run teach you containers in production without three
  weeks on the orchestrator. Learn k8s when a job needs it.
- **Microservices.** You'll build a well-structured monolith. Almost every team that
  started with microservices wishes they hadn't.
- **GraphQL.** REST first. Add GraphQL when you feel the pain it solves.
- **SSR frameworks.** A Vite SPA plus a real API keeps the client/server boundary
  visible.

---

## How long this takes

Honestly: **four to six months** at a normal working pace to get through all ten epics.
The API and frontend epics move fastest; the cloud epics feel slowest because you're
learning a vendor's vocabulary alongside the concept.

You are not behind if you're slower than that. You are behind if nobody knows you're
stuck — see [asking for help](06-asking-for-help.md).

---

**Next:** [Common stacks](02-common-stacks.md) — the map this stack sits in
