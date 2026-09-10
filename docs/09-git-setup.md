# Git setup

Set git up once, properly, so it stops costing you time. Most of this is a twenty-minute
job you'll benefit from for years.

**Time:** ~30 minutes
**You'll need:** a source control account <!-- TODO(team): which host? -->

---

## 1. Tell git who you are

Git stamps every commit with a name and email. Get this wrong and your commits show up
under the wrong identity — or worse, your personal email ends up in a public work repo.

```bash
git config --global user.name "Your Name"
git config --global user.email "your.work@email"
```

If you use one machine for work and personal projects, **don't set a global email at all.**
Set it per repo instead, so git errors out rather than guessing:

```bash
git config --global user.useConfigOnly true   # refuse to guess an email
cd ~/work/mirai && git config user.email "your.work@email"
cd ~/personal/thing && git config user.email "you@personal.com"
```

Check what a repo will actually use:

```bash
git config user.email
```

## 2. SSH keys

### Why SSH rather than HTTPS

You'll push many times a day. HTTPS means a token you have to store and rotate; SSH means
a key that just works. Set it up once.

### Generate a key

```bash
ssh-keygen -t ed25519 -C "your.work@email"
```

- **Use ed25519**, not RSA. Shorter, faster, and stronger.
- **Set a passphrase.** An unencrypted private key is a plaintext password lying on your
  disk. `ssh-agent` means you type it once per session, not once per push.
- Accept the default location unless you're managing several keys — see below.

Two files appear:

| File | What it is |
| --- | --- |
| `~/.ssh/id_ed25519` | **Private.** Never leaves this machine. Never pasted anywhere |
| `~/.ssh/id_ed25519.pub` | **Public.** This is the one you paste into the web UI |

> **If you ever paste the file without `.pub`** — anywhere, including a chat window,
> including to a colleague — that key is compromised. Generate a new pair and say so
> immediately. This is a normal mistake and a five-minute fix if you speak up.

### Load it into the agent

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

On macOS, add it to the keychain so it survives a reboot:

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

On Linux, most desktop environments start an agent for you; if `ssh-add -l` says the agent
isn't running, add the `eval` line to your shell profile.

### Add the public key to your account

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the whole line — it starts `ssh-ed25519` and ends with your email — and paste it into
your account's SSH keys settings. Name it after the machine (`work-laptop`), so you can
revoke a lost one without guessing.

**One key per machine.** Don't copy a private key between computers. If a laptop is lost,
you revoke that one key and everything else keeps working.

### Verify

```bash
ssh -T git@<!-- TODO(team): git host -->
```

You should see a greeting with your username. If you don't, see troubleshooting below.

### Several accounts on one machine

A common cause of "it says I don't have access" when you obviously do: git used your
personal key for a work repo. Fix it with host aliases in `~/.ssh/config`:

```
Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
    IdentitiesOnly yes

Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal
    IdentitiesOnly yes
```

Then clone using the alias instead of the real host:

```bash
git clone git@github-work:org/mirai.git
```

`IdentitiesOnly yes` matters: without it, ssh offers every key it has and the server
accepts the first one that works — which may not be the one you meant.

### Troubleshooting

| Symptom | Cause | Fix |
| --- | --- | --- |
| `Permission denied (publickey)` | Key not registered, or not offered | `ssh -T` to check; confirm you pasted the `.pub` |
| Works in one terminal, not another | Agent isn't running there | `eval "$(ssh-agent -s)" && ssh-add` |
| `UNPROTECTED PRIVATE KEY FILE` | Permissions too open | `chmod 600 ~/.ssh/id_ed25519` |
| Right key exists but wrong one is used | Multiple keys offered | `IdentitiesOnly yes` in `~/.ssh/config` |
| Asks for a passphrase every time | Key not in the agent | `ssh-add`, plus keychain on macOS |

To see exactly which key is being offered:

```bash
ssh -vT git@<host> 2>&1 | grep -i "offering\|accepted"
```

## 3. Sign your commits

Anyone can set `user.email` to yours and commit as you. Signing proves a commit came from
you. You can reuse the SSH key you already have — no GPG needed:

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

Then add the same public key to your account **again**, this time as a *signing* key —
it's a separate list from authentication keys, and forgetting that is the usual confusion.

<!-- TODO(team): is commit signing required, or optional? -->

## 4. Settings worth having

```bash
git config --global init.defaultBranch main
git config --global pull.ff only          # refuse a surprise merge commit on pull
git config --global push.autoSetupRemote true   # no more `--set-upstream`
git config --global rebase.autostash true       # stash and restore around a rebase
git config --global diff.colorMoved zebra       # show moved code distinctly in diffs
git config --global rerere.enabled true         # remember conflict resolutions
git config --global core.editor "code --wait"   # or vim, nano, whatever you use
```

Two of those repay themselves quickly:

- **`pull.ff only`** makes `git pull` fail rather than silently creating a merge commit
  when your branch has diverged. The failure tells you something you needed to know.
- **`rerere.enabled`** remembers how you resolved a conflict, so hitting the same one
  during a long rebase is automatic the second time.

## 5. Never commit

Some things can't be un-committed — once pushed, they're in the history on someone else's
machine, and deleting them in the next commit changes nothing.

- `.env` files, keys, tokens, passwords, certificates
- `node_modules`, `.venv`, `__pycache__`, build output
- Large binaries, database dumps, personal data
- Editor settings that are yours alone

The repo's `.gitignore` covers the common ones. For your personal ignores, use a global
file so you're not adding them to every project:

```bash
git config --global core.excludesfile ~/.gitignore_global
```

> **If you commit a secret**, say so immediately, before you try to fix it. The key must
> be rotated — removing it from history is not enough, because it has already been
> distributed. Nobody will be angry. A leaked secret nobody knows about is the actual
> problem.

---

## Checklist

- [ ] `git config user.email` shows the right address in a work repo
- [ ] `ssh -T` to the git host greets you by name
- [ ] Your key has a passphrase, and the agent holds it
- [ ] The key is named after this machine in your account settings
- [ ] If you have work and personal accounts, `~/.ssh/config` separates them
- [ ] `.gitignore` and your global excludes are in place

---

**Next:** [Git workflow](10-git-workflow.md) — branches, commits, and pull requests
