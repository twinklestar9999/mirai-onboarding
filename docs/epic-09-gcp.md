# Epic 09 — GCP

**MIRAI-080 – 086** · Deploy the same application to Google Cloud.

This epic is shorter than the AWS one on purpose. You already know the *concepts* — a
private network, a managed database, a container runtime, a CDN, a secret store. What
you're learning here is that they're the same concepts with different names, different
defaults, and different sharp edges.

That realisation is the entire point of doing it twice. After this epic, a third cloud
is a week, not a quarter.

> **Same cost warning as [epic 08](epic-08-aws.md).** Budget alerts before resources,
> Terraform for everything, tear down when you're done, check billing daily.

---

### MIRAI-080 — Project, budget, and IAM
**Size:** M · **Depends on:** —

**Acceptance criteria**
- [ ] A dedicated GCP project — the isolation boundary here, where AWS uses accounts
- [ ] Budget with alerts at 50%, 80%, 100% of the agreed cap
- [ ] Only the APIs you need are enabled (they're off by default — that's a real
      difference from AWS)
- [ ] A service account per workload, each least-privilege
- [ ] MFA on human accounts; nobody uses the project owner role day to day
- [ ] Labels on everything so spend is attributable

**You'll learn** — projects vs accounts, GCP's IAM model (roles bind to members on a
resource, rather than policies attached to identities), and API enablement.

**Hints** — write down every place GCP's model differs from AWS as you hit it. MIRAI-086
asks you for that list.

---

### MIRAI-081 — Terraform for GCP
**Size:** M · **Depends on:** MIRAI-080, MIRAI-071

**Goal.** A second Terraform root module, reusing what you can from the AWS one.

**Acceptance criteria**
- [ ] GCS backend for state, with versioning enabled
- [ ] Provider and version pinning
- [ ] Cloud-agnostic pieces factored into shared modules; cloud-specific pieces separate
- [ ] `terraform plan` clean
- [ ] The PR is honest about what actually turned out to be reusable — usually less than
      you'd hope, and knowing *why* is the lesson

**You'll learn** — multi-cloud Terraform structure, and where the abstraction genuinely
holds versus where "cloud agnostic" is marketing.

---

### MIRAI-082 — Cloud SQL
**Size:** M · **Depends on:** MIRAI-081

**Acceptance criteria**
- [ ] Postgres 17 with a private IP only — no public IP
- [ ] Automated backups and point-in-time recovery configured
- [ ] Connection via the Cloud SQL Auth Proxy or a private VPC connector
- [ ] Password generated into Secret Manager, never in Terraform code
- [ ] Deletion protection on
- [ ] The PR compares tier and monthly cost against your RDS instance

**You'll learn** — how GCP's connection model differs (an auth proxy rather than
security-group networking), and that "equivalent" instances price differently.

---

### MIRAI-083 — Cloud Run
**Size:** L · **Depends on:** MIRAI-082, MIRAI-067

**Goal.** The FastAPI container running on Cloud Run.

**Acceptance criteria**
- [ ] Image in Artifact Registry, deployed to Cloud Run
- [ ] Scales to zero, with min instances set for production to control cold starts
- [ ] Concurrency tuned deliberately — and the PR says why that number
- [ ] Dedicated service account with only the permissions it needs
- [ ] Alembic migrations run as a Cloud Run **job**, not on container start
- [ ] Health checks configured; a failing revision doesn't receive traffic
- [ ] Gradual traffic migration between revisions demonstrated

**You'll learn** — serverless containers, cold starts, request concurrency, and
revision-based traffic splitting — which is genuinely nicer than the ECS equivalent.
Note where GCP wins; being able to compare honestly is the skill.

---

### MIRAI-084 — Static hosting and attachments
**Size:** M · **Depends on:** MIRAI-083

**Acceptance criteria**
- [ ] GCS bucket for the web build, fronted by Cloud CDN and a global load balancer
- [ ] HTTPS with a Google-managed certificate
- [ ] SPA fallback to `index.html`
- [ ] Cache invalidated on deploy
- [ ] Separate private bucket for attachments, accessed via signed URLs
- [ ] Uniform bucket-level access on; no ACLs
- [ ] Public access prevention enforced at the organization level

**You'll learn** — GCP's load balancer is a bigger, more assembled component than an ALB.
Notice how many more pieces you wire up, and what you get for it.

---

### MIRAI-085 — Secrets and deploy from CI
**Size:** L · **Depends on:** MIRAI-084

**Acceptance criteria**
- [ ] All secrets in Secret Manager, mounted at runtime
- [ ] **Workload Identity Federation** from GitHub Actions — no service account JSON
      keys anywhere, ever
- [ ] Pipeline: test → build → push → `terraform apply` → deploy → smoke test
- [ ] Production deploy gated on manual approval
- [ ] Rollback demonstrated by shifting traffic to the previous revision
- [ ] You have grepped the build logs to confirm no secret is printed

**You'll learn** — keyless CI authentication on a second provider. A downloaded service
account key is a long-lived credential in a file, which is exactly what you spent
MIRAI-078 avoiding on AWS.

---

### MIRAI-086 — Compare the two, then tear down
**Size:** M · **Depends on:** MIRAI-085

**Goal.** The written comparison that makes this whole epic worth it.

**Acceptance criteria**
- [ ] `docs/cloud-comparison.md` covering, for each: developer experience, time to first
      deploy, cost for this workload, IAM model, networking model, docs quality, and the
      thing that surprised you most
- [ ] A concrete recommendation for a hypothetical team, with reasoning — and a note on
      what would change your answer
- [ ] Cost comparison with real numbers from your own two bills
- [ ] `terraform destroy` on both dev environments, verified clean
- [ ] Both consoles checked for orphans afterwards

**You'll learn** — how to evaluate a platform rather than repeat opinions about it. You
now have a first-hand answer to "AWS or GCP?", which is more than most people asking it
have.

**Hints** — resist writing a tie. Take a position and defend it. You can be wrong; you
can't be vague.

---

**Milestone check:** the same product, live on two clouds, at real URLs, deployed by CI
from the same repo. Show someone.

**Next:** [Epic 10 — Vercel](epic-10-vercel.md)
