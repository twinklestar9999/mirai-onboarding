# Epic 05 — Frontend

**MIRAI-042 – 055** · Turn the API into the site people actually use.

This epic ends at **Milestone M2**: a real, usable social site in a browser.

> **Before MIRAI-046, sketch the screens.** See [design with Claude](08-design-with-claude.md)
> — including the empty, loading, error and no-permission states, which are the ones that
> get built badly when they get designed last.

---

### MIRAI-042 — App shell and routing
**Size:** M · **Depends on:** MIRAI-006, MIRAI-032

**Acceptance criteria**
- [ ] Routes: `/`, `/explore`, `/c/:slug`, `/@:handle`, `/p/:postId`, `/notifications`,
      `/search`, `/settings`, `/login`, `/signup`, `/404`
- [ ] Logged-out visitors see Explore, not a login wall — a social site nobody can look at
      has no way to gain users
- [ ] Protected routes redirect to login and return to the intended page afterwards
- [ ] Persistent shell: nav, search field, notification badge
- [ ] Route-level code splitting

**You'll learn** — client-side routing, and that the logged-out experience is a product
decision, not an afterthought.

---

### MIRAI-043 — API client with generated types
**Size:** M · **Depends on:** MIRAI-004, MIRAI-028

**Goal.** A typed client where the TypeScript compiler catches a mismatch with the Python
API.

**Acceptance criteria**
- [ ] Request and response types come from the generated `types.gen.ts` (MIRAI-004) — no
      hand-written interfaces duplicating the Pydantic models
- [ ] Errors parsed into typed errors, not raw strings
- [ ] Credentials included so session cookies are sent
- [ ] Renaming a Pydantic field, then regenerating, breaks the frontend build

**You'll learn** — end-to-end type safety. That last criterion is the payoff for MIRAI-004.

---

### MIRAI-044 — Data fetching, by hand
**Size:** M · **Depends on:** MIRAI-043

**Goal.** Load the feed with `useEffect` and `useState`. Deliberately the hard way.

**Why.** You need to feel the problems before the library that solves them makes sense.

**Acceptance criteria**
- [ ] Loading, error, empty, and success states all handled
- [ ] Requests cancelled on unmount, with no state-update-after-unmount warning
- [ ] Race condition handled: switching fast between Home and Explore must not render the
      wrong feed
- [ ] The PR lists every problem you hit

**You'll learn** — effects, cleanup, stale closures, request races. Keep the list; the next
ticket deletes this code.

---

### MIRAI-045 — Replace it with TanStack Query
**Size:** M · **Depends on:** MIRAI-044

**Acceptance criteria**
- [ ] The hand-rolled fetching is gone
- [ ] Query keys structured and consistent
- [ ] Caching, background refetch, and retry configured deliberately
- [ ] Mutations invalidate the right queries — voting on a post updates it everywhere it
      appears on screen, not just where you clicked
- [ ] The PR maps each problem from MIRAI-044 to how the library solves it

**You'll learn** — server state vs client state, cache invalidation, and *why* a library
exists. That's more valuable than the library itself.

---

### MIRAI-046 — Design system foundation
**Size:** M · **Depends on:** MIRAI-042

**Acceptance criteria**
- [ ] shadcn/ui set up; Button, Input, Dialog, Dropdown, Avatar, Toast in place
- [ ] Design tokens as CSS variables — colors, spacing, radii
- [ ] Light and dark themes, following the OS by default with a manual override
- [ ] Keyboard focus visible everywhere
- [ ] Contrast meets WCAG AA, checked with a tool rather than by eye

**You'll learn** — tokens, theming, and that accessibility is cheap now and expensive to
retrofit.

---

### MIRAI-047 — Auth screens
**Size:** M · **Depends on:** MIRAI-046, MIRAI-031

**Acceptance criteria**
- [ ] Signup, login, forgot-password, reset-password
- [ ] Handle availability checked as they type, debounced, with the reserved list respected
- [ ] React Hook Form + Zod, mirroring the Pydantic rules for instant feedback (the server
      still validates — client validation is UX, never a security control)
- [ ] Field-level server errors render next to the right field
- [ ] Submit disabled while in flight; double-submit impossible
- [ ] Real `<label>`s and correct `autocomplete` attributes

**You'll learn** — form state, accessible forms, and optimistic availability checks.

---

### MIRAI-048 — The feed
**Size:** L · **Depends on:** MIRAI-045, MIRAI-023

**Goal.** The home screen — the one people will spend all their time on.

**Acceptance criteria**
- [ ] Infinite scroll on the cursor pagination from MIRAI-024
- [ ] Post cards: author, avatar, community, time, body, media, vote control, reply count
- [ ] Long posts truncate with "show more" rather than dominating the feed
- [ ] Skeleton placeholders while loading — no layout shift when content arrives
- [ ] Empty state for a new account that tells them what to do, with communities to join
- [ ] A "new posts" pill appears rather than shifting content under the reader
- [ ] 500 posts scrolled without the page becoming sluggish — virtualise if needed

**You'll learn** — infinite scroll, list virtualisation, and that inserting content above
someone's scroll position is one of the most disliked things a site can do.

---

### MIRAI-049 — The composer
**Size:** L · **Depends on:** MIRAI-048

**Acceptance criteria**
- [ ] Write a post: text, community picker, image attachment
- [ ] `@mention` autocomplete, keyboard navigable
- [ ] Character count that warns before the limit rather than truncating at it
- [ ] Draft preserved if they navigate away or reload
- [ ] Optimistic: the post appears in the feed immediately, marked pending, and reverts
      with a retry option if it fails
- [ ] Double-submit impossible

**You'll learn** — optimistic updates, draft persistence, and rollback UX. Losing a
half-written post is the fastest way to lose a user.

---

### MIRAI-050 — Threaded comments
**Size:** XL · **Depends on:** MIRAI-049, MIRAI-022

**Goal.** The post page — a nested conversation that stays readable at depth.

**Why.** The hardest frontend ticket. Recursive rendering, per-node state, and partial
loading all at once.

**Acceptance criteria**
- [ ] Comments render nested, with visual depth indication
- [ ] Collapse and expand any subtree, and the state survives a re-render
- [ ] "Load more replies" fetches a branch on demand
- [ ] Beyond max depth, replies flatten with a "continue this thread" link
- [ ] Reply inline at any level, optimistically
- [ ] Deleted comments show `[deleted]` and keep their children visible
- [ ] Readable at 375px width — nesting must not push text into a column two words wide
- [ ] A 1,000-comment thread renders without freezing the browser

**You'll learn** — recursive components, per-node UI state, and progressive loading. The
375px criterion is the one that kills naive indentation.

---

### MIRAI-051 — Profiles and communities
**Size:** L · **Depends on:** MIRAI-048

**Acceptance criteria**
- [ ] Profile: avatar, bio, counts, follow button, tabs for posts and replies
- [ ] Community: header, description, rules, member count, join button, its posts
- [ ] Follow and join are optimistic and idempotent — double-clicking is harmless
- [ ] Your own profile shows edit controls; nobody else's does
- [ ] Blocked users' profiles show a block state, not a crash
- [ ] Moderator controls appear only for moderators

**You'll learn** — conditional UI by permission, and that hiding a control is presentation
while the server check from MIRAI-034 is the actual security.

---

### MIRAI-052 — Image upload
**Size:** L · **Depends on:** MIRAI-049

**Acceptance criteria**
- [ ] Drag-and-drop or click, with a progress bar
- [ ] Direct-to-storage upload via a presigned URL — the file never passes through the API
- [ ] Type and size validated client-side *and* server-side
- [ ] Client-side downscale before upload — a 12MP phone photo must not be sent whole
- [ ] Preview before posting; remove before posting
- [ ] Failed uploads retryable, leaving no orphaned records
- [ ] Images render with correct aspect ratio and reserved space, so the feed doesn't jump

**You'll learn** — presigned URLs, client-side image processing, and why routing large
files through your API is how you run out of memory in production.

---

### MIRAI-053 — Search and notifications
**Size:** M · **Depends on:** MIRAI-027, MIRAI-026

**Acceptance criteria**
- [ ] Debounced search — no request per keystroke
- [ ] Tabbed results: posts, people, communities, with the matched term highlighted
- [ ] Keyboard navigable: arrows, Enter, Escape
- [ ] A slow search never overwrites the results of a later, faster one
- [ ] Notification list, grouped, unread-first, with an unread badge
- [ ] Opening the list marks read; the badge clears without a reload

**You'll learn** — debouncing, out-of-order response handling, and unread-state UX.

---

### MIRAI-054 — Live updates
**Size:** L · **Depends on:** MIRAI-050, MIRAI-026

**Goal.** New notifications and replies arrive without a reload.

**Acceptance criteria**
- [ ] Server-sent events stream notifications and new replies on an open thread
- [ ] Reconnects automatically with backoff after a drop
- [ ] Remote changes merge into the cache without clobbering a local optimistic update
- [ ] New content on screen is announced quietly, never by moving what someone is reading
- [ ] Connections close on unmount and on logout
- [ ] Tested by having someone else reply while you watch

**You'll learn** — SSE vs WebSockets, connection lifecycle, and merge conflicts between
local and remote state.

---

### MIRAI-055 — Polish and accessibility audit
**Size:** M · **Depends on:** MIRAI-054

**Acceptance criteria**
- [ ] Every route works keyboard-only, start to finish — including voting and collapsing
      comment threads
- [ ] Screen reader announces route changes and new content
- [ ] axe reports no violations
- [ ] Error boundaries — one bad post must not blank the whole feed
- [ ] Works at 375px, including deep comment threads
- [ ] `prefers-reduced-motion` respected
- [ ] Images have alt text, and the composer asks for it

**You'll learn** — that accessibility is a set of checkable requirements, not a vague
virtue. The composer asking for alt text is a product decision with a real user behind it.

---

**Milestone M2.** Give it to someone who has never seen it, say nothing, and watch. Write
down every place they hesitate. That list is worth more than any code review.

**Next:** [Epic 06 — Testing](epic-06-testing.md)
