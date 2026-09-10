# Setup

By the end of this page you'll have both toolchains installed and the project running
locally, with a change you made visible on your own screen.

**Time:** ~2 hours, mostly waiting on installs
**You'll need:** repo access <!-- TODO(team): who grants it, and how? -->

---

## 1. Accounts and access

Start these first — they often take a day — then install tools while you wait.

| What | Why | How to request |
| --- | --- | --- |
| Source control | Clone the repo, open PRs | <!-- TODO(team) --> |
| Chat | Where questions get answered | <!-- TODO(team): tool + channel --> |
| Ticket tracker | Where MIRAI tickets live | <!-- TODO(team) --> |
| AWS sandbox | Epic 08 | <!-- TODO(team) --> |
| GCP project | Epic 09 | <!-- TODO(team) --> |

> **If that fails:** if you don't have access after two days, say so in chat. Waiting
> quietly on access is the most common way a first week gets wasted.

## 2. The tools

Two toolchains, because the backend is Python and the frontend is TypeScript.

| Tool | Version | Install |
| --- | --- | --- |
| **uv** | latest | `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| **Python** | 3.13 | `uv python install 3.13` — uv manages it, don't use the system Python |
| **Node.js** | 22 LTS | via `fnm` or `nvm`, not a system package |
| **pnpm** | 9+ | `corepack enable && corepack prepare pnpm@latest --activate` |
| **Docker** | latest | Docker Desktop, or Docker Engine + Compose on Linux |
| **just** | latest | `cargo install just`, or your package manager |
| **git** | 2.40+ | your package manager |

**Never install project dependencies into your system Python.** `uv` creates a `.venv`
per project; that isolation is what stops two projects breaking each other.

Verify:

```bash
uv --version && node --version && pnpm --version && docker --version && just --version
```

## 3. SSH for source control

You'll push many times a day, so set this up once properly rather than typing a password
forever. The full walkthrough — keys, passphrase, the agent, several accounts on one
machine, and what to do when it says `Permission denied` — is in
[git setup](09-git-setup.md). Do that page now, then come back.

The short version:

```bash
ssh-keygen -t ed25519 -C "your.work@email"   # use a passphrase
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub                    # paste this into your account settings
ssh -T git@<!-- TODO(team): git host -->     # should greet you by name
```

> **Never paste the file *without* `.pub`.** That one is the private key. Pasting it
> anywhere means generating a new pair and saying so.

## 4. Clone and run

```bash
git clone <!-- TODO(team): repo URL -->
cd mirai

cp .env.example .env               # then fill in the values — see step 5
just install                       # uv sync + pnpm install
docker compose up -d               # Postgres, Redis, Mailpit
just db-migrate                    # apply Alembic migrations
just db-seed                       # sample data
just dev                           # API on :8000, web on :5173
```

You'll know it worked when:

- <http://localhost:5173> shows the app
- <http://localhost:8000/docs> shows the interactive API docs
- <http://localhost:8000/health> returns `{"status":"ok"}`

> **If that fails:** paste the *full* error into the template in
> [asking for help](06-asking-for-help.md) and post it. Setup breakage is almost never
> your fault — it's usually a missing step on this page, and telling us lets us fix it
> for the next person.

Note that until you've completed epic 01, most of the above doesn't exist yet — you're
building it. Come back and update this page as you go.

## 5. Secrets

<!-- TODO(team): how do new developers get local secrets — a shared vault, a teammate,
     a generated dev set? -->

Three rules, not negotiable:

- **Never commit a `.env`, key, token, or password.** Once it's in git history it is
  compromised, even if you delete it in the next commit.
- **Never paste a secret into chat**, including a DM, including "just temporarily."
- **If you leak one, say so immediately.** Rotating a key takes minutes. A leaked key
  nobody knows about is a genuine incident. Nobody will be angry — telling us fast is
  the entire job here.

## 6. Editor

Whatever you like, but configure these — they save hours:

- **Ruff** and **mypy** for Python; format on save
- **Biome** for TypeScript; format on save
- The Python interpreter pointed at the project's `.venv`, not the system one
- The repo's `.editorconfig` respected

<!-- TODO(team): is there a shared editor config or recommended extension set? -->

## 7. Make a change and see it

Don't skip this — it's the difference between "installed" and "working."

1. Change some visible text in the web app. Reload; confirm you see it.
2. Change a message in an API response. Confirm it at `/docs`.
3. Undo both: `git checkout .` — confirm they're gone.

You now have a working loop: edit, see the result, throw it away. That loop is most of
the job.

---

## Checklist

- [ ] Every access request submitted
- [ ] Both toolchains installed at the pinned versions
- [ ] `ssh -T` to the git host greets you by name
- [ ] Docker services up; migrations and seed applied
- [ ] Both apps running; `/docs` and the web app load
- [ ] You made a visible change on each side, saw it, and reverted it

---

**Next:** [How we work](05-how-we-work.md) — async, remote, and what's expected of you
