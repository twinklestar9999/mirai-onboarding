# Epic 08 — AWS

**MIRAI-070 – 079** · Deploy the app to AWS, entirely through Terraform.

> ## Read this before provisioning anything
>
> **Cloud costs real money, and the expensive mistakes are the quiet ones** — a load
> balancer left running, a NAT gateway you forgot, a database in a region you don't
> check. None of them error. They just bill.
>
> - MIRAI-070 sets up budget alerts and a hard cap. **Do it first.** No exceptions.
> - Every resource is created by Terraform, so `terraform destroy` can remove it.
>   Anything you click into existence in the console is something you will forget.
> - Tear down at the end of each week if you're not actively using it.
> - Check the billing dashboard every day this epic. Every day.
>
> <!-- TODO(team): sandbox account or shared? per-junior budget cap? who to tell if a
>      bill spikes? -->

---

### MIRAI-070 — Account, budgets, and IAM
**Size:** M · **Depends on:** —

**Goal.** An account you can't accidentally spend a fortune in.

**Acceptance criteria**
- [ ] Budget alerts at 50%, 80%, 100% of the agreed cap, emailing you
- [ ] MFA on every account that can log in
- [ ] You work as an IAM user or SSO role — **never** the root account
- [ ] Root account credentials stored securely and not used again
- [ ] A `mirai` cost allocation tag applied to everything, so you can see your own spend
- [ ] Region chosen deliberately and written down

**You'll learn** — the IAM model, least privilege, why root is for emergencies only.

---

### MIRAI-071 — Terraform foundations
**Size:** L · **Depends on:** MIRAI-070

**Goal.** Terraform with remote state, before it manages anything important.

**Why.** State is how Terraform knows what exists. Local state on your laptop means
nobody else can change anything, and losing your laptop orphans the infrastructure.

**Acceptance criteria**
- [ ] S3 backend with versioning, plus DynamoDB (or S3 native) state locking
- [ ] Provider and Terraform versions pinned
- [ ] Modules for reusable pieces; `dev` and `prod` as separate state
- [ ] `terraform plan` is clean and readable
- [ ] `README` explains the workflow: plan, review, apply
- [ ] No secrets in `.tf` files, and `.tfstate` is gitignored

**You'll learn** — declarative infrastructure, state, locking, and why you read a plan
before applying it. `plan` is a code review for your infrastructure.

---

### MIRAI-072 — Networking
**Size:** L · **Depends on:** MIRAI-071

**Goal.** A VPC where the database is unreachable from the internet.

**Acceptance criteria**
- [ ] VPC with public and private subnets across two availability zones
- [ ] Internet gateway for public; NAT for private egress
- [ ] Security groups referencing each other rather than IP ranges
- [ ] The database is in a private subnet and has no public IP
- [ ] The PR includes a network diagram
- [ ] The PR notes the monthly cost of the NAT gateway — it's the surprise on most bills

**You'll learn** — subnets, routing, security groups vs NACLs, and the habit of asking
"what could reach this?" about every resource.

---

### MIRAI-073 — RDS Postgres
**Size:** M · **Depends on:** MIRAI-072

**Acceptance criteria**
- [ ] Postgres 17 in private subnets, reachable only from the app's security group
- [ ] Automated backups with a defined retention window
- [ ] Encryption at rest and in transit
- [ ] The app requires TLS to connect
- [ ] Master password generated and stored in Secrets Manager, never in Terraform code
- [ ] Deletion protection on for prod
- [ ] The PR notes the instance size and monthly cost

**You'll learn** — managed databases, backup windows, encryption, and what a managed
service does and doesn't do for you.

---

### MIRAI-074 — ECR and ECS Fargate
**Size:** XL · **Depends on:** MIRAI-073, MIRAI-067

**Goal.** The API container running in the cloud.

**Acceptance criteria**
- [ ] ECR repository with a lifecycle policy so old images don't accumulate
- [ ] ECS cluster, task definition, and service on Fargate
- [ ] Task role scoped to exactly what the app needs
- [ ] Logs to CloudWatch, with a retention policy set (logs bill forever otherwise)
- [ ] Health check configured; unhealthy tasks are replaced
- [ ] At least 2 tasks across 2 AZs
- [ ] Migrations run as a one-off task before deploy, not on container start

**You'll learn** — container orchestration, task vs execution roles, and why running
migrations on every container start breaks the moment you scale past one.

---

### MIRAI-075 — Load balancer and HTTPS
**Size:** L · **Depends on:** MIRAI-074

**Acceptance criteria**
- [ ] ALB in public subnets routing to the ECS service
- [ ] ACM certificate; HTTP redirects to HTTPS
- [ ] Target group health checks hitting `/health`
- [ ] TLS 1.2 minimum, modern cipher policy
- [ ] Domain in Route 53 <!-- TODO(team): which domain do juniors get? -->
- [ ] Deploys cause no dropped requests, verified under load

**You'll learn** — load balancing, TLS termination, certificate validation, connection
draining.

---

### MIRAI-076 — Static hosting and attachments
**Size:** L · **Depends on:** MIRAI-075

**Acceptance criteria**
- [ ] S3 bucket for the web build, served via CloudFront
- [ ] Bucket is **private**; CloudFront reaches it via Origin Access Control
- [ ] SPA fallback to `index.html` for client routes
- [ ] Cache invalidated on deploy, and only for what changed
- [ ] Separate bucket for attachments, private, accessed only via presigned URLs
- [ ] Public access block on at the account level
- [ ] CORS configured for direct browser uploads

**You'll learn** — CDN behaviour, origin access, and that "public S3 bucket" is one of
the most common real-world data breaches.

---

### MIRAI-077 — Secrets
**Size:** M · **Depends on:** MIRAI-076

**Acceptance criteria**
- [ ] Every secret in Secrets Manager or Parameter Store
- [ ] Injected into the task at runtime; not in the image, not in the task definition
- [ ] IAM allows each service only the secrets it needs
- [ ] Rotation documented, and the database password rotated once as a drill
- [ ] No secret appears in CloudWatch logs, and you grepped to confirm

**You'll learn** — secret lifecycle, and that rotation only works if someone has done it
once before it's urgent.

---

### MIRAI-078 — Deploy from CI
**Size:** L · **Depends on:** MIRAI-077

**Goal.** Merge to `main` deploys to AWS, with no long-lived credentials anywhere.

**Acceptance criteria**
- [ ] GitHub OIDC federation — **no** static AWS access keys in GitHub
- [ ] Pipeline: test → build → push → `terraform apply` → deploy → smoke test
- [ ] Terraform plan posted on the PR for review before merge
- [ ] Production deploy requires manual approval
- [ ] Automatic rollback when the smoke test fails
- [ ] You have deployed, broken it deliberately, and watched it roll back

**You'll learn** — keyless CI authentication, and that a deploy pipeline nobody has seen
fail is untested.

---

### MIRAI-079 — Cost review and teardown
**Size:** M · **Depends on:** MIRAI-078

**Acceptance criteria**
- [ ] A cost breakdown by service in the PR, with the monthly projection
- [ ] The three most expensive line items identified, with one way to reduce each
- [ ] `terraform destroy` runs cleanly on the dev environment
- [ ] Console checked afterwards for orphans — Terraform doesn't know about anything you
      clicked, and snapshots and volumes often survive
- [ ] The rebuild is verified by applying from scratch

**You'll learn** — cloud economics, and that being able to destroy and rebuild is the
proof your infrastructure is genuinely code.

---

**Next:** [Epic 09 — GCP](epic-09-gcp.md)
