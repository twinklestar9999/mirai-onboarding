# Epic 10 — Vercel

**MIRAI-087 – 092** · Deploy the same frontend a third way, and learn what a platform
buys you and what it costs.

You've now done this the hard way twice. This epic is the contrast: a platform that
does most of epics 07–09 for you in about twenty minutes.

The point isn't that Vercel is easier — of course it is. The point is being able to say
*exactly* what it's doing on your behalf, what it charges for that, and where it stops
being the right answer. That's a judgement you can only make having built the underneath
yourself, which is why this epic is here and not at the start.

---

### MIRAI-087 — Deploy the frontend
**Size:** S · **Depends on:** MIRAI-069

**Goal.** The React app live on Vercel, built from the repo.

**Acceptance criteria**
- [ ] Connected to the repo; production deploys from `main`
- [ ] Build settings target `apps/web` in the monorepo, and the build doesn't pull in
      the Python app
- [ ] Environment variables set per environment, and no secret among them — anything in
      a frontend build is public, and this is where people learn that the expensive way
- [ ] SPA routing works: deep links don't 404
- [ ] Custom domain with HTTPS <!-- TODO(team): which domain? -->
- [ ] The PR records how long the whole thing took, start to finish

**You'll learn** — platform-as-a-service, and the difference between a build-time and a
runtime environment variable. `VITE_`-prefixed values are baked into the bundle and
readable by anyone who opens devtools.

**Hints** — compare that timing against epic 08. Sit with the difference; both numbers
are real, and so is what each one bought.

---

### MIRAI-088 — Preview deployments
**Size:** M · **Depends on:** MIRAI-087

**Goal.** Every pull request gets its own URL.

**Why.** This is Vercel's genuinely best feature, and the one worth stealing regardless
of platform. A reviewer clicking a link beats a reviewer reading a diff and imagining it.

**Acceptance criteria**
- [ ] Preview deployment per PR, with the URL commented on the PR automatically
- [ ] Previews point at a staging API, never production data
- [ ] Playwright from MIRAI-060 runs against the preview URL in CI
- [ ] Previews are protected — not publicly indexable, and not open to anyone with a link
      <!-- TODO(team): is preview access restricted to the team? -->
- [ ] The PR describes how you'd build the same thing on AWS, and roughly what it'd cost

**You'll learn** — ephemeral environments, and why "click here to try it" changes how
review actually works.

---

### MIRAI-089 — Edge functions and their limits
**Size:** M · **Depends on:** MIRAI-088

**Goal.** Move something to the edge, and find out what you can't do there.

**Acceptance criteria**
- [ ] One genuinely useful edge function — auth redirect, feature-flag routing, or
      geo-based defaults
- [ ] Latency measured from more than one region, before and after
- [ ] The PR lists the runtime's actual constraints you hit: no native modules, execution
      time limits, no direct TCP to your database
- [ ] An honest verdict on whether this was worth it for this use case

**You'll learn** — edge vs origin compute, cold starts at the edge, and that the reason
you can't just run your API there is the database connection, not the language.

**Hints** — the honest verdict is the deliverable. "It was faster but not usefully so"
is a perfectly good answer and a better one than pretending otherwise.

---

### MIRAI-090 — Connect it to a real backend
**Size:** M · **Depends on:** MIRAI-089, MIRAI-078

**Goal.** The Vercel frontend talking to your FastAPI backend on AWS or GCP.

**Why.** Vercel hosts your frontend beautifully and does not run your Python API. This is
the split-platform architecture most real teams end up with, and its problems are
different from either half.

**Acceptance criteria**
- [ ] CORS configured for the exact preview and production origins — not `*`, which would
      undo your cookie security
- [ ] Session cookies work cross-origin: `SameSite=None; Secure`, and the PR explains why
      that combination is required and what it gives up
- [ ] Preview deployments talk to staging; production talks to production, enforced by
      config rather than by remembering
- [ ] Latency measured for a full page load and compared against the single-cloud setup
- [ ] The PR names one thing this architecture makes harder

**You'll learn** — cross-origin authentication, which is where a large share of real
"it works locally" bugs live. Same-origin hides all of this from you.

---

### MIRAI-091 — Analytics and limits
**Size:** S · **Depends on:** MIRAI-090

**Acceptance criteria**
- [ ] Web Vitals collected from real users, not just a lab score
- [ ] Core Web Vitals measured: LCP, INP, CLS — with the current numbers in the PR
- [ ] One of them improved, with before/after
- [ ] The plan's actual limits written down: bandwidth, build minutes, function
      invocations, and what happens when you exceed each
- [ ] Cost projected at 10× your current traffic

**You'll learn** — real-user monitoring vs synthetic testing, and reading a pricing page
properly. Usage-based platform pricing is cheap until it very suddenly isn't.

---

### MIRAI-092 — The three-way comparison
**Size:** M · **Depends on:** MIRAI-091, MIRAI-086

**Goal.** Update `docs/cloud-comparison.md` from two platforms to three, and take a
position.

**Acceptance criteria**
- [ ] AWS, GCP and Vercel compared on: time to first deploy, ongoing operational effort,
      cost at low and at high traffic, control, lock-in, and what happens when it breaks
- [ ] **What Vercel is actually doing for you**, itemised against the epics you did by
      hand — CDN, TLS, CI, preview environments, rollback. Name the AWS or GCP service
      each replaces
- [ ] A recommendation for three different situations: a solo project, a funded startup,
      and a company with a compliance requirement — different answers, with reasoning
- [ ] The migration cost stated: how long to move off Vercel, and what you'd have to
      rebuild
- [ ] One thing you'd genuinely be sad to give up

**You'll learn** — platform evaluation and lock-in as a measurable cost rather than a
slogan. "Lock-in" means little until you can say how many weeks leaving would take.

**Hints** — resist the safe answer. Most engineers hold an opinion here they've never
had to justify with numbers. You now have the numbers.

---

**Milestone M4.** The same product live three ways — two clouds and a platform — at real
URLs, deployed by CI from one repo. And you can explain, without hedging, when you'd
choose each, and what you'd give up either way.

**Next:** [Epic 11 — Production](epic-11-production.md)
