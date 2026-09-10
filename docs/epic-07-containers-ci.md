# Epic 07 — Containers & CI/CD

**MIRAI-063 – 069** · Package the app so it runs anywhere, and automate the path from
merge to deployed.

Everything here is a prerequisite for both cloud epics. Do it once; AWS and GCP both
consume the output.

---

### MIRAI-063 — Dockerfile for the API
**Size:** M · **Depends on:** MIRAI-007

**Goal.** A small, secure production image.

**Acceptance criteria**
- [ ] Multi-stage build: dependencies, build, then a slim runtime stage
- [ ] Base image pinned by digest, not a floating tag
- [ ] Runs as a non-root user
- [ ] Built on `python:3.13-slim`; dependencies installed with `uv sync --frozen`
- [ ] `.dockerignore` excludes `.venv`, `__pycache__`, `.git`, `.env`, tests
- [ ] Final image under 300MB — report the size in the PR
- [ ] `HEALTHCHECK` defined
- [ ] Layers ordered so a source change doesn't reinstall dependencies

**You'll learn** — layer caching, multi-stage builds, and why running as root in a
container is a real finding in any security review.

---

### MIRAI-064 — Dockerfile for the web app
**Size:** M · **Depends on:** MIRAI-063

**Acceptance criteria**
- [ ] Build stage produces static assets; runtime stage serves them
- [ ] Client-side routes fall back to `index.html` (deep links must not 404)
- [ ] Security headers set: CSP, `X-Content-Type-Options`, `Referrer-Policy`
- [ ] Assets are content-hashed and cached long-term; `index.html` is not cached
- [ ] Gzip/Brotli enabled

**You'll learn** — SPA serving, cache headers, and why caching `index.html` ships a build
users can't escape.

---

### MIRAI-065 — Full stack in Compose
**Size:** M · **Depends on:** MIRAI-064

**Goal.** `docker compose up` runs the entire product: web, api, worker, Postgres, Redis,
Mailpit.

**Acceptance criteria**
- [ ] All services with health checks and correct `depends_on` conditions
- [ ] Migrations run automatically before the API starts
- [ ] A separate dev override file with hot reload
- [ ] A fresh clone reaches a working app with one command, verified by someone else
- [ ] The README documents every environment variable

**You'll learn** — service orchestration, startup ordering, and that "one command from
clone to running" is what makes onboarding the *next* person cheap.

---

### MIRAI-066 — Full test suite in CI
**Size:** L · **Depends on:** MIRAI-065, MIRAI-060

**Acceptance criteria**
- [ ] Unit, integration (Testcontainers), and e2e (Playwright) all run on PRs
- [ ] Jobs run in parallel where independent
- [ ] Playwright traces uploaded as artifacts on failure
- [ ] Full run under 10 minutes — report the time
- [ ] Reliable: no flaky failures across ten consecutive runs

**You'll learn** — pipeline design, parallelism, and that a slow or flaky pipeline gets
ignored, which makes it worse than none.

---

### MIRAI-067 — Build and publish images
**Size:** M · **Depends on:** MIRAI-066

**Acceptance criteria**
- [ ] Images built and pushed on merge to `main`
- [ ] Tagged with the commit SHA **and** a semver tag — never only `latest`
- [ ] Layer caching between builds
- [ ] Multi-arch (amd64 + arm64) if anyone develops on Apple silicon
- [ ] Scanned for vulnerabilities; the build fails on critical findings

**You'll learn** — registries, immutable tags, and why deploying `latest` means you can't
say what's running or roll back to what was.

---

### MIRAI-068 — Environment configuration
**Size:** M · **Depends on:** MIRAI-067

**Goal.** One image runs in any environment, configured entirely from outside.

**Acceptance criteria**
- [ ] All config from environment variables — no environment names in the code
- [ ] Config validated by a `pydantic-settings` model at import time; the app
      **refuses to boot** on missing or malformed config
- [ ] No secret in the image, the repo, or the build logs
- [ ] `.env.example` documents every variable and which are required
- [ ] Frontend config injected at runtime, not baked in at build time

**You'll learn** — twelve-factor config, and that failing loudly at startup beats
failing mysteriously an hour later.

**Hints** — the "refuses to boot" criterion matters. A missing secret should be a
startup crash with a clear message, not a 500 during a customer's checkout.

---

### MIRAI-069 — Release and rollback
**Size:** M · **Depends on:** MIRAI-068

**Goal.** A versioned, reversible release process.

**Acceptance criteria**
- [ ] Semantic versioning driven by Conventional Commits
- [ ] Changelog generated automatically
- [ ] A git tag per release, linked to the exact image
- [ ] A written, tested rollback procedure — you have actually rolled one back
- [ ] The PR states how long a rollback takes

**You'll learn** — release engineering, and that a rollback plan you've never executed
is not a rollback plan.

---

**Milestone M3.** Introduce a deliberate bug in a PR and watch CI block it. Then roll a
release back. Both should be boring.

**Next:** [Epic 08 — AWS](epic-08-aws.md)
