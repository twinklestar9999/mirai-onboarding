# Epic 11 — Production

**MIRAI-093 – 101** · Everything between "it's deployed" and "it's actually running a
business."

This is the epic that most self-taught developers never do, and it's the one that shows
up hardest in an interview. Anyone can deploy. Knowing what broke, at what time, for
which user, and being able to prove you fixed it — that's the job.

---

### MIRAI-093 — Structured logging
**Size:** M · **Depends on:** MIRAI-078

**Goal.** Logs you can query, not scroll.

**Acceptance criteria**
- [ ] JSON logs from the API — one object per line, no `print()`
- [ ] Every log line carries a request id, and it propagates through Celery tasks
- [ ] The request id is returned in a response header so a user can quote it
- [ ] Levels used correctly: `ERROR` means someone should look, not "something happened"
- [ ] Never logged: passwords, tokens, session ids, full request bodies, personal data
- [ ] Retention set in CloudWatch and Cloud Logging — logs bill forever otherwise
- [ ] You can answer "what happened to request X?" with a single query

**You'll learn** — structured logging, correlation ids, and log-level discipline. A
system where everything is `INFO` has no signal in it.

---

### MIRAI-094 — Distributed tracing
**Size:** L · **Depends on:** MIRAI-093

**Goal.** See where a slow request actually spent its time.

**Acceptance criteria**
- [ ] OpenTelemetry instrumentation for FastAPI, SQLAlchemy, and Celery
- [ ] Traces exported to the cloud provider's backend on both clouds
- [ ] Every database query appears as a span
- [ ] Trace id correlates with the log request id
- [ ] You used a trace to find one genuinely slow thing, and the PR shows before/after
- [ ] Sampling configured — tracing 100% of production traffic is expensive

**You'll learn** — spans, context propagation, sampling, and that "the API is slow" is a
symptom whose cause is usually a query you didn't know was running.

---

### MIRAI-095 — Metrics, dashboards, and alerts
**Size:** L · **Depends on:** MIRAI-094

**Acceptance criteria**
- [ ] The four golden signals: latency, traffic, errors, saturation
- [ ] p50/p95/p99 latency — **not** averages, and the PR explains why an average hides
      the users having the worst time
- [ ] A dashboard someone could read during an incident without a tour
- [ ] Alerts on symptoms users feel (error rate, latency), not on CPU
- [ ] Every alert links to a runbook saying what to do
- [ ] Alerts tuned so they don't fire spuriously — you tested by causing a real one
- [ ] A written SLO with an error budget

**You'll learn** — SRE fundamentals, percentiles, and alert fatigue. An alert that fires
weekly and is always ignored is worse than no alert.

---

### MIRAI-096 — Error tracking
**Size:** M · **Depends on:** MIRAI-093

**Acceptance criteria**
- [ ] Sentry on both the FastAPI backend and the React frontend
- [ ] Source maps uploaded so frontend stack traces are readable
- [ ] Release tagged, so an error points at the deploy that caused it
- [ ] Personal data scrubbed before sending
- [ ] Errors grouped sensibly, not one issue per user
- [ ] Alerting on new issues and on regressions, not every occurrence

**You'll learn** — error aggregation, release health, and privacy in observability
tooling — the tool that captures every exception will capture whatever was in scope.

---

### MIRAI-097 — Load testing and a performance budget
**Size:** L · **Depends on:** MIRAI-095

**Goal.** Find out where it breaks before a user does.

**Acceptance criteria**
- [ ] Load test with k6 or Locust against a production-like environment — never prod
- [ ] Realistic scenarios: scrolling the feed, opening a deep thread, posting, searching —
      weighted the way real traffic is, mostly reads
- [ ] The breaking point identified, and the PR names the actual bottleneck
- [ ] At least one bottleneck fixed, with before/after numbers
- [ ] A written performance budget: target p95 per endpoint
- [ ] A smoke-level load test in CI so a regression is caught

**You'll learn** — load vs stress vs soak testing, and that the bottleneck is almost
never where you guessed. Usually it's the database, and usually it's a missing index or
an N+1 that survived epic 02.

---

### MIRAI-098 — Security hardening
**Size:** L · **Depends on:** MIRAI-096

**Goal.** Review your own application the way an attacker would.

**Acceptance criteria**
- [ ] Security headers: CSP without `unsafe-inline`, HSTS, `X-Content-Type-Options`,
      `Referrer-Policy`
- [ ] Dependency scanning in CI for both `uv.lock` and `pnpm-lock.yaml`; criticals block
- [ ] Rate limiting on every mutating endpoint, not just login
- [ ] Upload validation by content inspection, not by trusting the file extension
- [ ] SQL injection impossible — parameterised queries throughout, verified by review
- [ ] The MIRAI-034 cross-visibility test suite re-run and still green
- [ ] A written threat model: what an attacker would want, and what stops them
- [ ] Container images scanned; base images updated

**You'll learn** — the OWASP Top 10 applied to code you wrote yourself, which is a very
different experience from reading the list.

**Hints** — CSP without `unsafe-inline` is the one that takes real work. Do it anyway;
it's the header that turns an XSS bug into a non-event.

---

### MIRAI-099 — Backup and restore drill
**Size:** M · **Depends on:** MIRAI-098

**Goal.** Prove you can get the data back. Not that backups exist — that a restore works.

**Why.** Untested backups fail at exactly the moment you need them. This ticket exists
because "we had backups" is a sentence people say during outages that don't end well.

**Acceptance criteria**
- [ ] Automated backups on both clouds, with a stated retention period
- [ ] **You restored one to a fresh database and started the app against it**
- [ ] Time-to-restore measured and written down
- [ ] Point-in-time recovery tested — restore to a timestamp five minutes ago
- [ ] A runbook with the exact commands, written for someone at 3am
- [ ] Stated RPO and RTO, and the PR says whether the measured times actually meet them

**You'll learn** — RPO/RTO, and the difference between a backup and a recovery plan.

---

### MIRAI-100 — Incident drill
**Size:** M · **Depends on:** MIRAI-099

**Goal.** Break production on purpose, respond, and write it up.

**Acceptance criteria**
- [ ] A teammate breaks something in your staging environment without telling you what
      <!-- TODO(team): who runs the game day for each junior? -->
- [ ] You detect it from monitoring, not from being told
- [ ] Time to detect, time to diagnose, and time to resolve all recorded
- [ ] A **blameless** postmortem: timeline, impact, root cause, contributing factors,
      action items with owners
- [ ] At least one action item implemented
- [ ] The five-whys goes past "someone made a mistake" to why the system allowed it

**You'll learn** — incident response, and blameless postmortem culture. The output of an
incident is a changed system, never a chastened person.

---

### MIRAI-101 — Handover and review
**Size:** M · **Depends on:** MIRAI-100

**Goal.** Leave the project in a state where someone else could take it over on Monday.

**Acceptance criteria**
- [ ] Architecture doc with a diagram, and the reasoning behind the main decisions
- [ ] Runbooks: deploy, rollback, restore, common alerts
- [ ] An updated README that a new developer can follow from clone to running
- [ ] `docs/decisions/` with short ADRs for the choices you'd have to defend
- [ ] A list of known issues and what you'd do next
- [ ] A 30-minute walkthrough delivered to the team

**You'll learn** — technical writing and architectural communication. The walkthrough is
the real assessment: explaining a system you built, out loud, to people who will ask
"why not the other way?" is the closest thing here to a senior-level conversation.

---

## You're done

Look back at [epic 01](epic-01-foundation.md). You could not have read that ticket list
in month one and known what half of it meant.

What you should now be able to do, and should say out loud in interviews:

- Design a relational schema and defend every constraint in it
- Build and secure an API, including the authorization cases most people get wrong
- Implement OAuth in all three directions — social login, third-party API access, and
  issuing scoped tokens — and say what `state` and PKCE each defend against
- Build an accessible frontend that handles latency honestly
- Test at every level, and know which level a given risk belongs to
- Provision infrastructure as code on two clouds, and say what a platform replaces
- Ship continuously, and roll back when it goes wrong
- Find out what broke, at what time, for which user — and prove you fixed it

The thing to keep from all of this isn't the stack — see
[common stacks](02-common-stacks.md) for how many others there are, and how much of what
you learned transfers. Stacks change. It's the habits:
reproduce before you fix, test the negative cases, read the plan before you apply, and
tell someone early when you're stuck.

**Back to:** [the board](12-ticket-board.md) · [the guide](../Readme.md)
