# Epic 04 — Auth

**MIRAI-029 – 041** · Build authentication and authorization by hand, replace it with a
provider, then do OAuth properly in all three directions.

Doing it by hand first is deliberate. Auth is the system you will most often be asked to
debug and least often be allowed to guess at. Build it once, understand it forever.

> **Read before starting.** Everything here follows published guidance — OWASP's
> Authentication and Session Management cheat sheets. When a ticket says "use the
> library," use the library. Hand-rolled *cryptography* is a different thing from
> hand-rolled *session plumbing*: you're learning the plumbing.

---

### MIRAI-029 — Password hashing
**Size:** M · **Depends on:** MIRAI-018

**Goal.** Register and verify passwords, stored as Argon2id hashes.

**Acceptance criteria**
- [ ] Argon2id via `argon2-cffi`, at current OWASP-recommended parameters
- [ ] Salt per password, handled by the library, never reused
- [ ] Plaintext passwords never logged, never stored, never returned
- [ ] Verification uses the library's `verify()`, which is constant-time — never `==`
- [ ] Minimum length enforced (12+); no composition rules
- [ ] Password field excluded from every API response by construction, not by remembering

**You'll learn** — why hashing isn't encryption (there's no decrypt), why bcrypt/Argon2
are deliberately slow, what a salt defends against, why timing matters.

**Hints** — "no composition rules" is deliberate: forced symbols produce `Password1!`.
Length beats character classes. Check the password against a known-breached list instead.

---

### MIRAI-030 — Server-side sessions
**Size:** L · **Depends on:** MIRAI-029

**Goal.** Sessions in the database, referenced by a cookie.

**Acceptance criteria**
- [ ] Session id from `secrets.token_urlsafe(32)` — never `random`, which is seeded
      and predictable
- [ ] Only a hash of the token is stored, so a database leak doesn't hand over live sessions
- [ ] Cookie is `HttpOnly`, `Secure`, `SameSite=Lax`, with a sensible `Max-Age`
- [ ] Absolute and idle expiry, both enforced server-side
- [ ] Logout deletes the session server-side, not just the cookie
- [ ] "Log out everywhere" deletes all of a user's sessions
- [ ] Expired sessions are cleaned up on a schedule

**You'll learn** — sessions vs JWTs (and why sessions are the right default: you can
revoke them), each cookie flag and the attack it blocks, why `HttpOnly` is what stops
a XSS bug from becoming an account takeover.

---

### MIRAI-031 — Signup, login, logout
**Size:** M · **Depends on:** MIRAI-030

**Acceptance criteria**
- [ ] `POST /auth/signup`, `/auth/login`, `/auth/logout`, `GET /auth/me`
- [ ] Login failures return one identical message and take the same time whether the
      email exists or not
- [ ] Session id is rotated on login (prevents session fixation)
- [ ] Signup on an existing email doesn't reveal that the account exists
- [ ] Tests cover: wrong password, unknown email, expired session, no session

**You'll learn** — user enumeration, session fixation, and why "no such user" vs "wrong
password" is a real information leak.

---

### MIRAI-032 — Auth middleware
**Size:** M · **Depends on:** MIRAI-031

**Goal.** Protected routes reject unauthenticated requests before reaching a handler.

**Acceptance criteria**
- [ ] Middleware loads the session and attaches the user, typed
- [ ] `401` with no valid session, `403` when authenticated but not permitted
- [ ] Routes are **protected by default** — a new route requires opting *out*, not in
- [ ] A test asserts every route is either explicitly public or protected

**You'll learn** — the difference between 401 and 403, and secure-by-default design.

**Hints** — the "protected by default" criterion is the important one. An auth system
where forgetting a line leaves a route open will eventually have an open route.

---

### MIRAI-033 — Organizations and membership
**Size:** L · **Depends on:** MIRAI-032

**Goal.** Users create communities, others join them, and members hold a role in each.

**Acceptance criteria**
- [ ] Create a community (creator becomes Owner, in one transaction — see MIRAI-017)
- [ ] Join and leave a public community; request-and-approve for a restricted one
- [ ] Roles from [the spec](03-project-spec.md): Owner, Moderator, Member
- [ ] Owners can promote and demote moderators; the last Owner cannot leave or be demoted
- [ ] Moderators can ban a user from their community, and a ban blocks posting, commenting
      and voting there — but never affects the rest of the site
- [ ] A banned user gets a clear reason, not a silent failure

**You'll learn** — role hierarchies, scoped moderation, and guarding against the state
where nobody can administer a community. Note that a ban is *scoped* — getting that wrong
means one moderator can remove someone from the whole site.

---

### MIRAI-034 — Authorization on every endpoint
**Size:** XL · **Depends on:** MIRAI-033

**Goal.** Every endpoint checks that *this* user may do *this* to *this specific row*.

**Why.** The most important ticket in the epic, and the most commonly botched thing in
real applications. Broken object-level authorization is consistently at the top of the
OWASP API risk list — usually as "authenticated user changes an id in the URL and reads
content they were never permitted to see."

**Acceptance criteria**
- [ ] A single `can(user, action, resource)` authorization layer — not scattered `if`s
- [ ] Every endpoint verifies community membership, role, and block state
- [ ] Every fetch by id is scoped to what the caller may see in the **query**, so a
      wrong id returns `404` rather than being fetched and then rejected
- [ ] A test suite that, for every endpoint, attempts access as: a non-member, a Viewer,
      a banned user, and a user who has been blocked by the author — and expects failure
- [ ] Missing and forbidden both return `404` where revealing existence would leak

**You'll learn** — object-level authorization, why scoping in the query beats checking
after the fetch, defense in depth.

**Hints** — write the negative tests first. If it's tedious, that's the point: this is
the test suite that stops you leaking another customer's data.

---

### MIRAI-035 — Rate limiting and lockout
**Size:** M · **Depends on:** MIRAI-034

**Goal.** Login and other sensitive endpoints resist automated guessing.

**Acceptance criteria**
- [ ] Redis-backed rate limiting, per IP and per account
- [ ] Progressive delay or temporary lockout after repeated failures
- [ ] `429` with a `Retry-After` header
- [ ] Lockout can't be used to deny a legitimate user their account indefinitely
- [ ] Limits are configurable per environment

**You'll learn** — credential stuffing, why per-IP alone is insufficient, and the
availability trade-off in lockout design.

---

### MIRAI-036 — Password reset
**Size:** L · **Depends on:** MIRAI-035

**Goal.** Forgotten-password flow by email, queued as a background job.

**Acceptance criteria**
- [ ] Token is random, hashed at rest, single-use, expires in ~1 hour
- [ ] The request endpoint responds identically whether the email exists or not
- [ ] Email sent via a Celery task with retries; the request doesn't wait on SMTP
- [ ] Resetting invalidates all existing sessions
- [ ] Local development catches mail with Mailpit instead of sending
      <!-- TODO(team): what's the real email provider in staging/production? -->

**You'll learn** — secure token design, queues, why a slow third party must never sit in
a request's critical path.

---

### MIRAI-037 — Replace it with an OIDC provider
**Size:** L · **Depends on:** MIRAI-036

**Goal.** Swap your hand-built auth for a hosted provider, keeping your own
authorization layer.

**Why.** The point of the epic lands here. You now know exactly what the provider is
doing for you, what it costs, and what you're handing over.

**Acceptance criteria**
- [ ] OIDC authorization-code flow with PKCE <!-- TODO(team): which provider? -->
- [ ] Existing users are migrated or linked, not orphaned
- [ ] Your `can()` layer from MIRAI-034 is unchanged — authorization stays yours
- [ ] A written comparison in the PR: what got simpler, what got harder, what you gave up

**You'll learn** — OAuth2/OIDC, PKCE, and the build-vs-buy judgement, which you can now
actually make rather than guess at.

---

### MIRAI-038 — Sign in with Google and GitHub
**Size:** L · **Depends on:** MIRAI-037

**Goal.** Social login, implemented properly — and account linking that doesn't create
duplicate users.

**Why.** OAuth is the flow you'll meet in nearly every product you work on, and it's
routinely implemented with a security hole in it. Doing it once, carefully, with the
spec open, is worth more than any tutorial.

**Acceptance criteria**
- [ ] Authorization code flow with PKCE for both providers
- [ ] The `state` parameter is random per request, stored server-side, and **verified on
      callback** — this is what stops CSRF on the login flow, and it's the check people skip
- [ ] Redirect URIs are exact-matched against an allowlist; no wildcards, no prefix matching
- [ ] Signing in with Google using the email of an existing password account **links** to
      it rather than creating a second user — after verifying the provider says the email
      is verified
- [ ] A user can see their linked accounts and unlink one, but not the last sign-in method
- [ ] Provider tokens are encrypted at rest; the app's own session is still yours
- [ ] Tests cover: user denies consent, provider returns an error, callback replayed,
      `state` mismatch

**You'll learn** — the authorization code flow end to end, what `state` and PKCE each
defend against (they are different attacks), and why "sign in with X" is an
*authentication* claim you have to decide whether to trust.

**Hints** — read the flow in RFC 6749 §4.1 and the OAuth 2.0 Security Best Current
Practice document. Both are shorter than you expect. The account-linking criterion is the
one with a real vulnerability behind it: linking on an unverified email lets someone
register `you@example.com` at a sloppy provider and take over your account.

---

### MIRAI-039 — Act on a user's behalf with a third-party API
**Size:** L · **Depends on:** MIRAI-038

**Goal.** Let a user connect an external social account and cross-post to it.
<!-- TODO(team): which platform — Mastodon, X, Bluesky? Pick one with a usable free API. -->

**Why.** Social login only reads an identity. This is the other half of OAuth: holding a
token and using it later, which means refresh, expiry, and revocation — the parts that
break in production.

**Acceptance criteria**
- [ ] Incremental scopes — request calendar access when the user connects it, not at signup
- [ ] Refresh tokens stored encrypted; access tokens never logged
- [ ] Automatic refresh on expiry, with the refresh handled once under concurrency rather
      than by every in-flight request at the same time
- [ ] Revocation handled: if the user disconnects at the provider, your app notices and
      asks them to reconnect instead of failing silently
- [ ] Rate limits and provider outages handled with backoff — a third party being down
      must not take a page of your app down with it
- [ ] Disconnecting deletes the stored tokens and calls the provider's revocation endpoint

**You'll learn** — token lifecycle, incremental authorization, and defensive integration
with a service you don't control. The concurrent-refresh criterion is a real production
bug: ten requests refreshing at once, nine of them invalidating each other's tokens.

---

### MIRAI-040 — Issue scoped API tokens
**Size:** M · **Depends on:** MIRAI-039

**Goal.** Users can generate personal access tokens so scripts and bots can use Mirai's API.

**Why.** You've been the OAuth *client* twice. This is the provider side — deciding what
a token may do and how it's revoked. It's also what makes your API usable by anything
that isn't a browser.

**Acceptance criteria**
- [ ] Tokens are random and stored hashed — shown to the user exactly once, at creation
- [ ] Scopes: `read`, `write`, `admin`, enforced through the same `can()` layer from
      MIRAI-034, not a parallel permission system
- [ ] Optional expiry; `last_used_at` recorded so users can spot a stale token
- [ ] Revocable, and revocation takes effect immediately
- [ ] Token auth accepted via `Authorization: Bearer`, alongside session cookies
- [ ] A token can never exceed the permissions of the user who created it, and there's a
      test proving it
- [ ] Rate limits apply per token, not just per user

**You'll learn** — scoped authorization, credential display-once patterns, and supporting
two authentication mechanisms without duplicating the authorization logic.

---

### MIRAI-041 — Auth security review
**Size:** M · **Depends on:** MIRAI-040

**Goal.** Audit everything in this epic against the OWASP guidance, and write down what
you found.

**Acceptance criteria**
- [ ] Reviewed against the OWASP Authentication, Session Management, and Authorization
      cheat sheets — a written checklist with pass/fail per item
- [ ] Verified: `state` checked, redirect URIs exact-matched, tokens hashed at rest,
      sessions rotated on privilege change, no token in a URL or a log
- [ ] Every auth-related error message checked for information leakage
- [ ] The cross-visibility suite from MIRAI-034 re-run against token auth as well as
      session auth — a new authentication path is a new way to get authorization wrong
- [ ] `docs/auth.md` explaining the whole design to a future maintainer, with a diagram
      of each flow
- [ ] Anything you couldn't fix is written up as a known issue with its risk

**You'll learn** — how to audit auth rather than hope. Also: writing your own design down
is the fastest way to notice what's wrong with it.

---

**Milestone check:** ask a teammate to try to read a restricted community they're not in,
post as someone else, and vote after being banned — through the API, through the UI, and
with an API token. If any of it works, MIRAI-034 isn't done.

**Next:** [Epic 05 — Frontend](epic-05-frontend.md)
