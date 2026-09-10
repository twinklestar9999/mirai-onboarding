# Epic 01 — Foundation

**MIRAI-001 – 008** · Set up the repo everything else is built in.

This epic feels like admin work. It isn't — a repo where a Python model change breaks
the TypeScript build is what makes the next nine epics fast. Get it right once.

---

### MIRAI-001 — Create the repo skeleton
**Size:** S · **Depends on:** —

**Goal.** `apps/api` (Python) and `apps/web` (TypeScript), each installable from the
root with one command.

**Why.** Backend and frontend in one repo means one clone, one branch, one PR per
feature — and the contract between them can be checked automatically.

**Acceptance criteria**
- [ ] `apps/api/pyproject.toml` with uv, Python pinned to 3.13 via `.python-version`
- [ ] `apps/web/package.json`, with `pnpm-workspace.yaml` at the root
- [ ] `uv sync` and `pnpm install` both work from a documented path
- [ ] A `justfile` (or `Makefile`) wraps both: `just install`, `just dev`, `just check`
- [ ] `.gitignore` covers `node_modules`, `.venv`, `__pycache__`, `dist`, `.env`
- [ ] `README.md` explains how to install and run

**You'll learn** — project layout, virtual environments, why lockfiles are committed and
`.venv` never is.

**Hints** — the `justfile` matters more than it looks. Two toolchains means two of every
command, and one entry point is what stops that being annoying every single day.

---

### MIRAI-002 — Strict typing on both sides
**Size:** M · **Depends on:** MIRAI-001

**Goal.** `mypy --strict` on Python and `strict: true` on TypeScript, both passing.

**Why.** Python's type hints do nothing at runtime — mypy is what makes them real.
Turning either on later means fixing hundreds of errors at once.

**Acceptance criteria**
- [ ] mypy configured in `pyproject.toml` with `strict = true`, passing on all of `apps/api`
- [ ] `tsconfig.json` with `strict`, `noUncheckedIndexedAccess`, `noImplicitOverride`
- [ ] `just typecheck` runs both and passes
- [ ] No `Any`, no `# type: ignore`, no `any`, no `@ts-ignore` — and where one is truly
      unavoidable it carries a comment explaining why

**You'll learn** — gradual typing, why `Any` silently disables checking for everything
downstream of it, and why `noUncheckedIndexedAccess` catches a real class of crash.

---

### MIRAI-003 — Lint and format
**Size:** S · **Depends on:** MIRAI-001

**Goal.** `just format` fixes style everywhere; `just lint` reports what it can't fix.

**Why.** Formatting arguments in code review are a waste of everyone's time. A tool
settles them, permanently.

**Acceptance criteria**
- [ ] Ruff configured for both lint and format on Python
- [ ] Biome on TypeScript
- [ ] Line length and quote style agreed once and applied by the tools
- [ ] `pre-commit` runs both on staged files
- [ ] `just format` produces zero diff on a clean tree

**You'll learn** — linting vs formatting (they're different), and why automated style
beats a style guide nobody reads.

---

### MIRAI-004 — The API contract
**Size:** L · **Depends on:** MIRAI-002

**Goal.** Pydantic models on the backend generate TypeScript types the frontend imports.

**Why.** This is the keystone ticket. You define `Task` once, in Python. FastAPI emits an
OpenAPI schema; a generator turns it into TypeScript. Rename a field in Python and the
frontend stops compiling — which is exactly what you want, because the alternative is
finding out in production.

**Acceptance criteria**
- [ ] Pydantic models for at least `User` and `Task` in `apps/api`
- [ ] `just gen-types` exports the OpenAPI schema and runs `openapi-typescript`
- [ ] Generated types land in `apps/web/src/api/types.gen.ts`, committed, with a header
      marking them generated — never edit by hand
- [ ] The frontend imports and uses them
- [ ] **Renaming a Pydantic field breaks `pnpm typecheck` in `apps/web`**
- [ ] The README documents when to regenerate

**You'll learn** — schema-first design, code generation, and contract testing between
services.

**Hints** — the second-to-last criterion is the real test of this ticket. Do it before
you open the PR: rename a field, regenerate, confirm the frontend fails, then fix it.

---

### MIRAI-005 — Hello world API
**Size:** S · **Depends on:** MIRAI-002

**Goal.** A FastAPI app with `GET /health` returning `{"status": "ok"}`.

**Why.** The smallest thing that runs, so later tickets change something known to work
instead of debugging two things at once.

**Acceptance criteria**
- [ ] `just dev-api` starts uvicorn with auto-reload
- [ ] `curl localhost:8000/health` returns the JSON
- [ ] Interactive docs at `/docs` already work — look at them, that's Pydantic and
      FastAPI doing it for free
- [ ] Port and host come from environment variables with sensible defaults
- [ ] An unknown route returns JSON, not an HTML error page

**You'll learn** — ASGI, async handlers, and why an API returning HTML errors breaks
every client that isn't a browser.

---

### MIRAI-006 — Hello world web app
**Size:** S · **Depends on:** MIRAI-004

**Goal.** A Vite + React + Tailwind app that fetches `/health` and shows the result.

**Acceptance criteria**
- [ ] `just dev-web` serves with hot reload
- [ ] The page shows the API's status, **and an error state when the API is down**
- [ ] The response is typed using the generated types from MIRAI-004
- [ ] Tailwind working (verify with an obvious utility class)
- [ ] API URL from an env var, not a hard-coded string

**You'll learn** — Vite, the dev proxy, CORS (you'll meet it here — understand it rather
than pasting `allow_origins=["*"]`), and that the error state is part of the feature.

**Hints** — stop the API and reload. A blank screen or a console error means that
criterion isn't met.

---

### MIRAI-007 — One command for everything
**Size:** M · **Depends on:** MIRAI-003, MIRAI-005, MIRAI-006

**Goal.** `just check` runs typecheck, lint, and tests across both apps.

**Why.** This exact command is what CI runs and what reviewers assume passed. It needs to
be one thing you can run before every push.

**Acceptance criteria**
- [ ] `just check` = mypy + ruff + pytest + tsc + biome + vitest
- [ ] `just dev` starts API and web together
- [ ] Failures are readable — you can tell which tool failed and why
- [ ] `just check-fast` skips the slow parts for a tight loop

**You'll learn** — task orchestration across two ecosystems, and why one entry point is
what makes a check actually get run.

---

### MIRAI-008 — First CI pipeline
**Size:** M · **Depends on:** MIRAI-007, MIRAI-004

**Goal.** GitHub Actions runs `just check` on every PR and blocks merge on failure.

**Acceptance criteria**
- [ ] Triggers on pull requests to `main`
- [ ] Python, uv, Node and pnpm versions pinned, matching local
- [ ] Dependency caching for both ecosystems
- [ ] **A job regenerates the API types and fails if the committed ones differ** — this
      is what stops the contract silently drifting
- [ ] Branch protection requires the check <!-- TODO(team): who configures branch protection? -->
- [ ] You proved it works by opening a PR that deliberately fails, and watching it block

**You'll learn** — CI concepts, version pinning, and generated-code drift checks.

**Hints** — the drift check is the ticket's real content. Without it, MIRAI-004's
guarantee only holds when someone remembers to run the generator.

---

**Milestone check:** `just install && just check && just dev` works from a fresh clone.
Ask someone to clone your repo and run exactly those. If it fails on their machine the
epic isn't finished — and finding that out now is the entire point.

**Next:** [Epic 02 — Database](epic-02-database.md)
