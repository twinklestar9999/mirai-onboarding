# Design with Claude

How to design a screen before you build it, and why that's faster even when it feels
slower.

**Time:** ~15 minutes to read; use it throughout [epic 05](epic-05-frontend.md)

---

## Why design first

The usual junior approach to a UI ticket is to open the component file and start typing.
It feels productive. What actually happens is that layout decisions get made one `div` at
a time, the screen ends up as a pile of accumulated choices, and the third change breaks
the second one.

Designing first is not about making things pretty. It's about **making the decisions when
they're cheap.** Moving a box in a mockup takes ten seconds. Moving it in a built
component takes an hour, plus the responsive breakpoints, plus the tests.

You are also going to be asked, in review, why the screen works the way it does. Having
thought about it beforehand is the difference between an answer and a shrug.

---

## Claude Design

`/design` creates a **design canvas**: several artboards laid out on one pan-and-zoom
surface, published as a link you can share. You describe the screens; it drafts them; you
refine elements directly — click to select, edit text inline, adjust properties, undo.

For this program it's most useful for:

- **Screen flows** — signup → empty state → first board → task detail, seen side by side
- **Comparing options** before committing to one
- **States you'd otherwise forget** — loading, empty, error, permission-denied
- **Something to put in a PR** so a reviewer can react to a picture rather than imagine one

It is a drafting tool, not a design system. It won't tell you your contrast fails, it
won't enforce your tokens, and it doesn't know what shadcn/ui gives you for free. Those
stay your job.

---

## The screens Mirai needs

Before you start MIRAI-046, put these on a canvas. Doing all of them at once is the point
— the inconsistencies show up when they're next to each other:

| Screen | The states that matter |
| --- | --- |
| Home feed | New account with zero follows, normal, loading, end of feed |
| Post card | Short text, long text truncated, with image, deleted, from a blocked user |
| Post detail | Deep thread, collapsed branches, `[deleted]` parent, 1,000 replies |
| Composer | Empty, typing, mention autocomplete open, image attached, over limit, failed |
| Profile | Your own, someone else's, blocked, brand new with no posts |
| Community | Joined, not joined, restricted, banned |
| Notifications | Empty, unread, grouped |
| Search | Idle, typing, results, no results |

**The empty state is the screen every new user sees first**, and it's the one that gets
designed last, badly, if at all. Design it first instead — a social site that opens on an
empty feed loses the user in about four seconds.

---

## How to work with it

### Describe the content, not the decoration

Weak:

> design a nice modern social feed

Strong:

> A social feed screen for a community site. Post cards show author avatar and handle, the
> community name, relative time, body text, an optional image, a vote control with a score,
> and a reply count. A composer sits at the top. Show six posts — one long enough to
> truncate, one with an image, one from a community the user just joined. Light and dark.

The second gets you something you can actually evaluate, because it contains the real
constraints.

### Ask for the variants together

> Show three versions of the post card: compact, comfortable, and image-forward. Same data
> in each.

Choosing between three concrete things is much easier than critiquing one.

### Design the hard states, not the happy one

The happy path designs itself. Ask specifically for: a title long enough to wrap, a user
with no avatar, a board with one task, a failed save, a viewer who can't edit. That list
is where UI bugs come from, and it's what MIRAI-059's acceptance criteria will ask about.

### Then hand it to yourself

Once you're happy, the useful next prompt is:

> Break this into React components. Which are shadcn/ui primitives, which are new, and
> what props does each need?

That turns a picture into a build plan, and it's where you'll notice that four screens all
need the same card component.

---

## What stays your responsibility

A generated mockup is a starting point that has not been checked against any of this. From
[epic 05](epic-05-frontend.md)'s criteria:

- **Contrast meets WCAG AA.** Check with a tool. Generated palettes frequently fail, and
  it is never visible by eye until someone can't read it.
- **Keyboard access.** A mockup can't show focus order. Decide it deliberately — every
  interaction in MIRAI-050 and MIRAI-055 must work without a mouse.
- **375px width.** If you only design at desktop width, you'll discover the phone layout
  in review.
- **Real content.** Design with the longest name and the emptiest board you'll actually
  have, not with tidy sample data.
- **Your tokens.** The colours and spacing in the mockup mean nothing until they're the
  CSS variables you defined in MIRAI-046.

---

## If there's a designer

<!-- TODO(team): does the team have a designer, and what's the handoff process? -->

Then this changes: their designs are the source of truth, and your job is to build them
accurately and to raise the states they didn't cover — which is usually loading, error,
and empty. Use the canvas to *ask a question* ("did you mean this when the list is
empty?"), never to overrule the design.

Bring implementation constraints early and specifically. "That animation will cost us a
layout shift on every load" is useful in the design review and annoying afterwards.

---

## The habit worth keeping

Ten minutes of sketching before a UI ticket saves hours, every time, and it survives any
tool. What you're really practising is deciding what a screen is for before deciding what
it looks like.

---

**Next:** [Git workflow](09-git-workflow.md) — branches, commits, and pull requests
