# Epic 06 — Testing

**MIRAI-056 – 062** · Build the safety net that lets you change things without fear.

Tests are not homework. They're what makes the cloud epics survivable: you're about to
deploy this to two providers, and you need a way to know it still works that isn't
clicking around.

---

### MIRAI-056 — pytest and the first unit tests
**Size:** M · **Depends on:** MIRAI-007

**Goal.** Fast unit tests for pure logic — ordering, mention parsing, permissions.

**Acceptance criteria**
- [ ] `just test` runs pytest on the API and Vitest on the web app
- [ ] Tests for the MIRAI-023 ordering logic, mention parsing, and `can()`
- [ ] Edge cases covered: empty, single item, duplicates, boundaries
- [ ] `@pytest.mark.parametrize` used for table-driven cases instead of copy-pasted tests
- [ ] Test names describe behaviour — `test_viewer_cannot_edit_task`, not
      `test_permissions`
- [ ] No test touches the database or the network

**You'll learn** — arrange/act/assert, pytest fixtures and `parametrize`, and that a
test name is documentation read at 2am by whoever broke it.

---

### MIRAI-057 — Integration tests on a real database
**Size:** L · **Depends on:** MIRAI-056, MIRAI-017

**Goal.** Repository and service tests against real Postgres, via Testcontainers.

**Why.** Mocked databases pass while production breaks. Constraints, cascades and
transactions only exist in a real one — and they're exactly what you want tested.

**Acceptance criteria**
- [ ] Testcontainers spins up Postgres from a session-scoped fixture; Alembic
      migrations run automatically
- [ ] Each test is isolated — transaction rollback or truncation between tests
- [ ] Tests can run in parallel without interfering
- [ ] Covers cascade deletes, unique constraints, and transaction rollback
- [ ] The suite passes on a clean machine with only Docker installed

**You'll learn** — test isolation, and why shared mutable state is what makes a suite
flaky.

---

### MIRAI-058 — API tests, including the negative cases
**Size:** L · **Depends on:** MIRAI-057, MIRAI-034

**Goal.** Every endpoint tested through real HTTP, especially the ways it should fail.

**Acceptance criteria**
- [ ] Happy path for every endpoint
- [ ] Auth cases: no session, expired session, wrong role, banned from the community,
      blocked by the author
- [ ] Validation: missing fields, wrong types, oversized payloads
- [ ] Correct status codes and error shapes asserted, not just "not 200"
- [ ] The MIRAI-034 cross-visibility suite lives here permanently

**You'll learn** — that most valuable tests assert what *shouldn't* work. Anyone can
test the happy path; the bugs are elsewhere.

---

### MIRAI-059 — Component tests
**Size:** M · **Depends on:** MIRAI-047

**Goal.** React components tested the way a user experiences them.

**Acceptance criteria**
- [ ] Testing Library, queried by role and label — not by test id or class name
- [ ] MSW mocks the API at the network layer, so real fetch code runs
- [ ] Loading, error, and empty states all covered
- [ ] Form validation and submission tested through user events
- [ ] No test asserts on internal state or implementation details

**You'll learn** — testing behaviour over implementation. A test that breaks when you
rename a variable is a test that costs more than it saves.

---

### MIRAI-060 — End-to-end with Playwright
**Size:** L · **Depends on:** MIRAI-059, MIRAI-054

**Goal.** The critical journeys, in a real browser, against the real stack.

**Acceptance criteria**
- [ ] Journeys: sign up → join a community → post with an image → another user replies →
      vote → the reply appears in notifications
- [ ] Runs against Compose with a seeded database
- [ ] Auth state reused between tests instead of logging in every time
- [ ] Trace and screenshot captured on failure
- [ ] No arbitrary `waitForTimeout` — wait for conditions
- [ ] Runs in CI, headless, reliably ten times in a row

**You'll learn** — e2e trade-offs (slow and brittle, but the only thing that proves the
pieces fit) and why sleeps are the primary cause of flaky suites.

---

### MIRAI-061 — Coverage, and what not to test
**Size:** M · **Depends on:** MIRAI-060

**Goal.** Coverage reporting, plus a written testing strategy.

**Acceptance criteria**
- [ ] Coverage in CI, visible on the PR
- [ ] A threshold that fails the build, agreed rather than guessed
      <!-- TODO(team): what's the expected threshold? -->
- [ ] `docs/testing.md` explains what gets a unit vs integration vs e2e test
- [ ] It also lists what you deliberately *don't* test, and why

**You'll learn** — the testing pyramid, and that 100% coverage is a target that produces
worthless tests. Coverage tells you what *isn't* tested; it says nothing about whether
the tests are good.

---

### MIRAI-062 — Turn a bug into a test
**Size:** S · **Depends on:** MIRAI-058

**Goal.** Practise the regression loop on a real bug from your own project.

**Acceptance criteria**
- [ ] Pick a bug you actually hit in epics 01–05
- [ ] Write a failing test that reproduces it — **first**
- [ ] Fix it; the test goes green
- [ ] The PR shows the failing output, then the passing output

**You'll learn** — the single most useful habit in this epic. Reproduce, then fix. A fix
without a failing test first is a guess that happened to work.

---

**Milestone check:** deliberately break something — remove an auth check, invert a
condition — and confirm the suite catches it. A suite that never fails isn't protecting
you.

**Next:** [Epic 07 — Containers & CI/CD](epic-07-containers-ci.md)
